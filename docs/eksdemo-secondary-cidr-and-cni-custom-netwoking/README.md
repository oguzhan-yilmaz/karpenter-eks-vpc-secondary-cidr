# EKS Custom Newtworking w/ VPC Secondary CIDR Demo

- This configuration **keeps nodes and pods IP addresses in the different CIDR blocks**.
- AWS VPCs has a default CIDR block, and you can add a secondary CIDR block to the VPC.
- We will use the secondary CIDR block for the pods, and the default CIDR block of the VPC for the nodes.
  - Secondary CIDR will have 3 subnets with `/19` mask, so we can have available IP count of `3*8190~=24570` for our pods.
- This tutorial also includes karpenter configuration for make use of the secondary CIDR block.
- This demo is for pre `v0.32` or `v1alpha` Karpenter version, but should work fine for AWS CNI and ENIConfig.

## Why is this needed?

- Running many nodes in EKS can cause IP address exhaustion in the VPC.
- How many IP addresses are available to a node is determined by nodes ENI capacity.
    - Because of this, EKS requires running many nodes to keep up with the Pod count.
- Using a VPC with Secondary CIDR block allows us to have more IP addresses available to our pods.
- Karpenter is a faster option for cluster autoscaling than the default EKS Cluster Autoscaler.
- Karpenter can be configured to use Spot Instances, which can save a lot of money.


## Hands-on Demo

### Prerequisites

- `jq`
- `eksdemo`
- `yq`
- `kubectl`
- `aws cli`

### About the Demo

Following is a full demo that will **configure CNI Custom Networking for EKS**:

- create an EKS Cluster with a VPC Secondary CIDR block,
- and will configure AWS VPC CNI,
- and also the Karpenter to use the secondary CIDR block for the pods.

### Index

- [1. Create VPC with Secondary CIDR and Subnets](1-vpc-secondary-cidr-and-subnets.md)
- [2. AWS VPC CNI & ENIConfig configuration for Custom Networking
  ](2-aws-vpc-cni-configuration.md)
- [3. Karpenter v1alpha Configuration (Provider & AWSNodeTemplate)](3-karpenter-v1alpha-configuration.md)
