# EKS Custom Newtworking w/ VPC Secondary CIDR Demo

- This configuration **keeps nodes and pods IP addresses in the different CIDR blocks**.
- AWS VPCs has a default CIDR block, and you can add a secondary CIDR block to the VPC.
- We will use the secondary CIDR block for the pods, and the default CIDR block for the nodes.
  - Secondary CIDR will have `/19` mask so for 3 subnets we can have available IP count of `3*8190` or `24570` for our pods.
- This tutorial also includes karpenter configuration for make use of the secondary CIDR block.
- This demo is for pre `v0.32` or `v1alpha` Karpenter version, but should work fine for AWS CNI and ENIConfig.

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