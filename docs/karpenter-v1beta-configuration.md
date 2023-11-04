# Demo: Karpenter v1beta
https://github.com/awslabs/amazon-eks-ami/blob/master/files/bootstrap.sh

- karpenter.k8s.aws/instance-pods https://karpenter.sh/docs/reference/instance-types/
  - check this node label!
- https://github.com/awslabs/amazon-eks-ami/blob/master/log-collector-script/linux/README.md
  - collect logs of userdata + kubelet???

## Installation

```bash title="Export env. variables we will use in this demo"
export KARPENTER_VERSION=v0.32.1
export K8S_VERSION=1.24
export AWS_PAGER=""                          # disable the aws cli pager
export AWS_PROFILE=hepapi
export AWS_REGION=eu-central-1
export CLUSTER_NAME="bambi"                 # will be created with eksctl
```

### Create an EKS cluster with Karpenter

```bash
eksctl create cluster -f - <<EOF
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${CLUSTER_NAME}
  region: ${AWS_REGION}
  version: "${K8S_VERSION}"
  tags:
    karpenter.sh/discovery: ${CLUSTER_NAME} # here, it is set to the cluster name
iam:
  withOIDC: true 

karpenter:
  version: "${KARPENTER_VERSION}"
  createServiceAccount: true 
  defaultInstanceProfile: "KarpenterNodeInstanceProfile-${CLUSTER_NAME}"
  withSpotInterruptionQueue: true 

managedNodeGroups:
- instanceType: m5.large
  amiFamily: AmazonLinux2
  name: ${CLUSTER_NAME}-ng
  desiredCapacity: 2
  minSize: 1
  maxSize: 5
EOF
```

### Find all AWS resources tagged with `karpenter.sh/discovery=${CLUSTER_NAME}`

Check if you got everything tagged correctly.

It should include the following resources (just as your original EKS NodeGroup):

- Private Subnets
- Cluster SecurityGroup

```bash
# List all resources with the tag: 'karpenter.sh/discovery=${CLUSTER_NAME}'
aws resourcegroupstaggingapi get-resources \
    --tag-filters "Key=karpenter.sh/discovery,Values=${CLUSTER_NAME}" \
    --query 'ResourceTagMappingList[]' --output text \
    | sed 's/^arn:/\n----------\narn:/g'
```



```bash title="Check Karpenter logs"
kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter
```

- Maybe set hostNetwork: true ?

```bash title="Check interruped sqs queue"
aws sqs get-queue-attributes \
    --queue-url "https://sqs.${AWS_REGION}.amazonaws.com/${ACCOUNT_ID}/${CLUSTER_NAME}" \
    --attribute-names ApproximateNumberOfMessages --no-cli-pager --query 'Attributes'
```

### Scale up the cluster

```bash title="Scale up the cluster"



```