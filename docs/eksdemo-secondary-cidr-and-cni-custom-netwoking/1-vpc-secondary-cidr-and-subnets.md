# 1. Create VPC with Secondary CIDR and Subnets


### Export Variables

```bash title="Export the variables we will be using throughout the demo"
export AWS_PAGER=""                          # disable the aws cli pager 
export AWS_PROFILE=hepapi
export AWS_REGION=eu-central-1                   
export CLUSTER_NAME="tenten"                 # will be created with eksdemo tool
export CLUSTER_VPC_CIDR="194.151.0.0/16"     # main EKS Cluster VPC CIDR
export SECONDARY_CIDR_BLOCK="122.64.0.0/16"  # secondary CIDR block, will be used for pod IPs
export AZ1_CIDR="122.64.0.0/19"              # -> make sure to 
export AZ2_CIDR="122.64.32.0/19"             # -> use the correct
export AZ3_CIDR="122.64.64.0/19"             # -> AZ CIDR blocks and masks
export AZ1="eu-central-1a"                      
export AZ2="eu-central-1b"
export AZ3="eu-central-1c"
export NODEGROUP_NAME="main"                 # default is 'main', keep this value
```

### Create eksdemo EKS cluster

- Use `eksdemo` to create a PoC EKS cluster.

```bash title="Create an EKS Cluster with eksdemo or use your own cluster's context"
# this command can take up to 15 minutes to complete
# change the k8s version if you want to
eksdemo create cluster "$CLUSTER_NAME" \
    --instance "m5.large" \
    --nodes 1 \
    --version "1.24" \
    --os "AmazonLinux2" \
     --vpc-cidr "$CLUSTER_VPC_CIDR"

# get the cluster info from eksdemo
eksdemo get cluster "$CLUSTER_NAME"

# make sure to use the correct context
eksdemo use-context "$CLUSTER_NAME"

kubectl config current-context

kubectl get pod -A
kubectl get nodes -o wide
```

### Create Secondary VPC CIDR

#### Export VPC Info

- We need to export VPC related info to use in the following steps.

```bash
export VPC_ID=$(aws eks describe-cluster --name "$CLUSTER_NAME" --query "cluster.resourcesVpcConfig.vpcId" --output text)

echo "VPC ID: ${VPC_ID:-'ERROR: should have VPC_ID, fix before continuing'}"

export VPC_NAME=$(aws ec2 describe-vpcs --vpc-ids "$VPC_ID" --query 'Vpcs[].Tags[?Key==`Name`].Value' --output text)

echo "VPC_NAME: ${VPC_NAME:-'ERROR: should have VPC_NAME, fix before continuing'}"

export CLUSTER_SECURITY_GROUP_ID=$(aws eks describe-cluster --name "$CLUSTER_NAME" --query cluster.resourcesVpcConfig.clusterSecurityGroupId --output text)
echo "EKS Cluster($CLUSTER_NAME) has Security Group ID: ${CLUSTER_SECURITY_GROUP_ID:-'ERROR: should have CLUSTER_SECURITY_GROUP_ID, fix before continuing'}"

```

#### Associate Secondary CIDR Block to VPC

```bash 
echo "\nCurrent Subnets:"
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].{SubnetId: SubnetId,AvailabilityZone: AvailabilityZone,CidrBlock: CidrBlock}' \
    --output table

# Associate a secondary CIDR block with the cluster VPC
echo "\nAssociating secondary CIDR block: $SECONDARY_CIDR_BLOCK with VPC: $VPC_NAME($VPC_ID)"
aws ec2 associate-vpc-cidr-block --vpc-id "$VPC_ID" --cidr-block "$SECONDARY_CIDR_BLOCK" --no-cli-pager


# see the created associations as a table
echo "\n$VPC_NAME($VPC_ID) VPC has associations:"
aws ec2 describe-vpcs --vpc-ids "$VPC_ID" \
    --query 'Vpcs[*].CidrBlockAssociationSet[*].{CIDRBlock: CidrBlock, State: CidrBlockState.State}' --out table

```

#### Create Subnets in the Secondary CIDR Block

Create 3 subnets in the Secondary CIDR block with `/19` mask, so we can have available IP count of `3*8190` or `24570` for our pods.

```bash title="Set CUSTOM_SNET{1,2,3} variables if you've already created the subnets"
# Control the variables before creating the subnets
echo "Provided Vars (for later export):\nexport AZ1=$AZ1\nexport AZ2=$AZ2\nexport AZ3=$AZ3\nexport AZ1_CIDR=$AZ1_CIDR\nexport AZ2_CIDR=$AZ2_CIDR\nexport AZ3_CIDR=$AZ3_CIDR\n"

# create the subnets
echo "Creating Subnets on AZs: $AZ1, $AZ2 and $AZ3"
export CUST_SNET1=$(aws ec2 create-subnet --cidr-block "$AZ1_CIDR" --vpc-id "$VPC_ID" --availability-zone "$AZ1" | jq -r .Subnet.SubnetId)
export CUST_SNET2=$(aws ec2 create-subnet --cidr-block "$AZ2_CIDR" --vpc-id "$VPC_ID" --availability-zone "$AZ2" | jq -r .Subnet.SubnetId)
export CUST_SNET3=$(aws ec2 create-subnet --cidr-block "$AZ3_CIDR" --vpc-id "$VPC_ID" --availability-zone "$AZ3" | jq -r .Subnet.SubnetId)

echo -e "Created Subnets:\nexport CUST_SNET1=$CUST_SNET1 # ($AZ1_CIDR)\nexport CUST_SNET2=$CUST_SNET2 # ($AZ2_CIDR)\nexport CUST_SNET3=$CUST_SNET3 # ($AZ3_CIDR)"
# do this if associated to Public Route table ( or to an internet gateway)
# Enable auto-assign public IPv4 addresses
# aws ec2 modify-subnet-attribute --subnet-id "$CUST_SNET1" --map-public-ip-on-launch 
# aws ec2 modify-subnet-attribute --subnet-id "$CUST_SNET2" --map-public-ip-on-launch 
# aws ec2 modify-subnet-attribute --subnet-id "$CUST_SNET3" --map-public-ip-on-launch 
```

```bash
echo "VPC Subnets after Secondary CIDR has been populated with 3 subnets:"
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].{SubnetId: SubnetId,AvailabilityZone: AvailabilityZone,CidrBlock: CidrBlock}' \
    --output table
```


#### Route Table Configuration

- Our subnets must have an egress to an `InternetGateway` or a `NatGateway`, so we need to configure the route tables.
- We will use the auto-generated Route Table for our Secondary CIDR block, and we will add a route to the NAT Gateway.

```bash title="You can skip this step if you handled the route table configuration manually"
# Find the RouteTableID of the main route table
export MAIN_ROUTE_TABLE_ID=$(aws ec2 describe-route-tables \
    --filters "Name=route.destination-cidr-block,Values=${SECONDARY_CIDR_BLOCK}"  "Name=association.main,Values=true" \
    --query 'RouteTables[].RouteTableId' --output text)

echo "Routes of RouteTable w/ ID: ${MAIN_ROUTE_TABLE_ID:-'MAIN_ROUTE_TABLE_ID variable should have a value of route-table-id, fix before continuing...'}"

aws ec2 describe-route-tables --route-table-ids "$MAIN_ROUTE_TABLE_ID" --query 'RouteTables[].Routes[]' --output table


# not really needed, bc aws is smart enough to group our secondary CIDR subnets in a route table
#   aws ec2 associate-route-table --route-table-id $MAIN_ROUTE_TABLE_ID --subnet-id $CUST_SNET1
#   aws ec2 associate-route-table --route-table-id $MAIN_ROUTE_TABLE_ID --subnet-id $CUST_SNET2
#   aws ec2 associate-route-table --route-table-id $MAIN_ROUTE_TABLE_ID --subnet-id $CUST_SNET3

# Find a (the first) NATGW ID in the VPC
export FIRST_NATGW_ID=$(aws ec2 describe-nat-gateways \
    --filter "Name=vpc-id,Values=${VPC_ID}" \
    --query 'NatGateways[0].NatGatewayId' --output text 2>&1)

echo "Found (first) NATGW ID: ${FIRST_NATGW_ID:-'FIRST_NATGW_ID variable should have a value, fix before continuing...'}"
```

```bash
# create a route in the MAIN_ROUTE_TABLE_ID and at 0.0.0.0/0 with the FIRST_NATGW_ID. Description: "Route to secondary CIDR block"
echo "Creating NATGW route in RouteTable(${MAIN_ROUTE_TABLE_ID})"
natgw_route_has_created=$(aws ec2 create-route \
    --route-table-id "$MAIN_ROUTE_TABLE_ID" \
    --destination-cidr-block "0.0.0.0/0" \
    --nat-gateway-id "$FIRST_NATGW_ID" --output text)
echo "In RouteTable(${MAIN_ROUTE_TABLE_ID}), created a route at 0.0.0.0/0 to NATGW(${FIRST_NATGW_ID}): $natgw_route_has_created"

aws ec2 describe-route-tables --route-table-ids "$MAIN_ROUTE_TABLE_ID" --query 'RouteTables[].Routes[]' --output table

```

#### Tagging the Resources properly

- `Karpenter AWSNodeTemplate` objects select the Subnets and Security Groups based on the `karpenter.sh/discovery` tag.
    ```yaml title="A fragment of a Karpenter AWSNodeTemplate yaml"
    kind: AWSNodeTemplate
    spec:
        subnetSelector: 
            karpenter.sh/discovery: "${CLUSTER_NAME}"        
        securityGroupSelector: 
            karpenter.sh/discovery: "${CLUSTER_NAME}"
    ```

```bash title="Find the NodeGroup Subnets and tag them: karpenter.sh/discovery=${CLUSTER_NAME}"
# We need to keep the original EKS Node Group configuration
# regarding subnets. Our configuration tries to keep Nodes and Pods
# in different CIDR blocks, so we need to tag the existing subnets.
existing_node_group_subnets=$(aws eks describe-nodegroup \
  --cluster-name "${CLUSTER_NAME}" \
  --nodegroup-name "${NODEGROUP_NAME}" \
  --query 'nodegroup.subnets' \
  --output text | cat -s \
  | awk -F'\t' '{for (i = 1; i <= NF; i++) print $i}')

echo "Existing Node Group Subnets: \n${existing_node_group_subnets:-'ERROR: should have existing_node_group_subnets, fix before continuing'}"
```

- The tags that start with `kubernetes.io/*` are required.
- For `kubernetes.io/role/*` tag, follow the below rules:
    - Public Subnets: `kubernetes.io/role/elb,Value=1`
    - Private Subnets: `kubernetes.io/role/internal-elb,Value=1`

```bash title="Do the actual tagging after you've checked the output of the previous command"
while IFS=$'\t' read -r subnet_id ; do
    # echo "${subnet_id}"
    echo "Tagging Subnet: $subnet_id of EKS Cluster: $CLUSTER_NAME + NodeGroup: $NODEGROUP_NAME"
    aws ec2 create-tags --resources "$subnet_id" --tags \
    "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}"
done <<< $existing_node_group_subnets

```


```bash title="Tag the Subnets we created for the Secondary CIDR block + Cluster SG"
# tag the subnets
aws ec2 create-tags --resources "$CUST_SNET1" --tags \
    "Key=Name,Value=SecondarySubnet-A-${CLUSTER_NAME}" \
    "Key=kubernetes.io/role/internal-elb,Value=1" \
    "Key=kubernetes.io/cluster/${CLUSTER_NAME},Value=shared"

aws ec2 create-tags --resources "$CUST_SNET2" --tags \
    "Key=Name,Value=SecondarySubnet-B-${CLUSTER_NAME}" \
    "Key=kubernetes.io/role/internal-elb,Value=1" \
    "Key=kubernetes.io/cluster/${CLUSTER_NAME},Value=shared"

aws ec2 create-tags --resources "$CUST_SNET3" --tags \
    "Key=Name,Value=SecondarySubnet-C-${CLUSTER_NAME}" \
    "Key=kubernetes.io/role/internal-elb,Value=1" \
    "Key=kubernetes.io/cluster/${CLUSTER_NAME},Value=shared"

# tag Cluster Security Group as well 
# (NOTE: the tag "kubernetes.io/cluster/${CLUSTER_NAME}=shared" is required and is probably already there)
aws ec2 create-tags --resources "$CLUSTER_SECURITY_GROUP_ID" --tags \
    "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
    "Key=kubernetes.io/cluster/${CLUSTER_NAME},Value=owned"

```

### Next Steps

- [2. AWS VPC CNI & ENIConfig configuration for Custom Networking](2-aws-vpc-cni-configuration.md)
- [3. Karpenter v1alpha Configuration (Provider & AWSNodeTemplate)](3-karpenter-v1alpha-configuration.md)
