# 2. AWS VPC CNI & ENIConfig configuration for Custom Networking

### AWS-Node and CNI Configuration

```bash
# Get the current env vars of aws-node
kubectl get daemonset aws-node -n kube-system -o jsonpath='{.spec.template.spec.containers[0].env}' | jq -r '.[] | .name + "=" + .value'

# Enable custom network config in aws-node
kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true

kubectl set env daemonset aws-node -n kube-system ENI_CONFIG_LABEL_DEF=topology.kubernetes.io/zone

# restart aws eni controller
kubectl rollout restart ds/aws-node -n kube-system
```

#### Environment Variables Explained

- `kube-system ENI_CONFIG_LABEL_DEF=topology.kubernetes.io/zone`:
  - Means that AWS CNI is using the _topology.kubernetes.io/zone_ label to determine the `ENIConfig` name(`kubectl get eniconfig`) for that node.
  - _topology.kubernetes.io/zone_ label is automatically added to the nodes by the kubelet as `eu-west-1a` or `eu-west-1b` or `eu-west-1c`, so we don't need any extra node tagging to do.
  - This way we have a consistent way of applying the ENIConfig to the nodes.
  - `ENIConfig` has the info about which Subnet and Security Groups should be used for the ENI.
  - Our nodes will have their 1st ENI configured with the default VPC CIDR block, and the 2nd ENI will be configured with the Secondary CIDR block.
  - Pods get their IPs from 2nd ENI, and the 1st ENI is used for the node communication.
  - We will have 1st ENI reserved for Nodes, and all other ENIs will be used for the pod communication and in the Secondary CIDR block.
- `AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true`:
  - Enables the Custom Network Configuration for AWS CNI. This change will help us to use the secondary CIDR block for the pods.
  - AWS CNI will use the `ENIConfig` objects to create and configure the ENIs.
  - AWS CNI will look for the label `${ENI_CONFIG_LABEL_DEF}` on the node, and will use the value of that label to find the `ENIConfig` object by name.
  - This configuration **requires the existing node EC2 Instances be be restarted to take effect**.

#### More on ENIConfig CRDs

- ENIConfig CRD is used by AWS CNI to create ENIs with the specified configuration for that Availability Zone.
- The deamonset `aws-node` has env. var. called `ENI_CONFIG_LABEL_DEF`, and it is used to match

  ```
  NodeLabels:
      topology.kubernetes.io/zone=eu-west-1a
      ...

  AWS CNI makes the following configuration
      (selected ENIConfig name for node) = NodeLabels[ENI_CONFIG_LABEL_DEF]
  ```

- We are informing AWS CNI to look for the node label `topology.kubernetes.io/zone`.
  - For example, if the label value is `eu-west-1a`, AWS CNI will use the `ENIConfig` named `eu-west-1a`.

### Let's create the ENIConfig objects

- It's crucial to use AZ name as the ENIConfig name.
  - This is because the `aws-node` deamonset uses the `ENI_CONFIG_LABEL_DEF` env. var. to match the node label value to the `ENIConfig` name.

```bash
cat << EOF | kubectl apply -f -
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
 name: $AZ1
spec:
  securityGroups: ["$CLUSTER_SECURITY_GROUP_ID"]
  subnet: "$CUST_SNET1"
EOF

cat << EOF | kubectl apply -f -
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
 name: $AZ2
spec:
  securityGroups: ["$CLUSTER_SECURITY_GROUP_ID"]
  subnet: "$CUST_SNET2"
EOF

cat << EOF | kubectl apply -f -
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
 name: $AZ3
spec:
  securityGroups: ["$CLUSTER_SECURITY_GROUP_ID"]
  subnet: "$CUST_SNET3"
EOF

kubectl get eniconfig ${AZ1} -o yaml; echo "---";
kubectl get eniconfig ${AZ2} -o yaml; echo "---";
kubectl get eniconfig ${AZ3} -o yaml; echo "---";
```

### Restart the Node Group Instances

- Terminate the Node Group instances to have them recreated with the new ENI configuration.
  - Node should get it's primary ENI from the default VPC CIDR block, and the secondary ENI(and any other ENIs) from the Secondary CIDR block.
  - This is because we have configured the ENIConfig objects to use the Secondary CIDR block subnets.
- After you recreate the Node instances, **check the instances to see if they got their IP Addresses from VPC Secondary CIDR Block**
- You should be seeing 1st ENI with IP from the default VPC CIDR block, and others from the Secondary CIDR block
  - ![Managed Node EC2 Instance should have ips](../images/managed-node-instance-ip-addrs-on-secondary-cidr.png)


### Test Pods having IP addresses from Secondary CIDR Block

```bash
kubectl create deployment nginx --image=nginx
kubectl scale --replicas=5 deployments/nginx
kubectl expose deployment/nginx --type=NodePort --port 80

# try to see if the pods are running on the secondary CIDR block 
# (note: ignore daemonset pods)
kubectl get pods -o wide

kubectl port-forward svc/nginx 8000:80
# check localhost:8000 on browser
```


### Next Steps

- [3. Karpenter v1alpha Configuration (Provider & AWSNodeTemplate)](3-karpenter-v1alpha-configuration.md)
- [Demo Troubleshooting](troubleshooting.md)