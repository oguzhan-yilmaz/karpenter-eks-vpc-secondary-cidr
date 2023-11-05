# karpenter-eks-vpc-secondary-cidr

Docs on creating an EKS setup with Secondary CIDR block for Pod IP addresses. Includes Karpenter configuration. 


This repo uses [eksdemo](https://github.com/awslabs/eksdemo) to create an EKS Cluster and install karpenter.

### [Go to Documentation Website](https://oguzhan-yilmaz.github.io/karpenter-eks-vpc-secondary-cidr/)

## Index

- **Demo: AWS VPC CNI Custom Networking**
    - [About the Demo](eksdemo-secondary-cidr-and-cni-custom-netwoking/README.md)
    - [1. Create VPC with Secondary CIDR and Subnets]()
    - [2. AWS VPC CNI & ENIConfig configuration for Custom Networking](eksdemo-secondary-cidr-and-cni-custom-netwoking/2-aws-vpc-cni-configuration.md)
    - [3. Karpenter v1alpha Configuration](eksdemo-secondary-cidr-and-cni-custom-netwoking/3-karpenter-v1alpha-configuration.md)
    - [Demo Troubleshooting](eksdemo-secondary-cidr-and-cni-custom-netwoking/troubleshooting.md)
- [ec2-instance-selector CLI tool](ec2-instance-selector.md)





## Why is this needed?

- Running many nodes in EKS can cause IP address exhaustion in the VPC.
- How many IP addresses are available to a node is determined by nodes ENI capacity.
    - Because of this, EKS requires running many nodes to keep up with the Pod count.
- Using a VPC with Secondary CIDR block allows us to have more IP addresses available to our pods.
- Karpenter is a faster option for cluster autoscaling than the default EKS Cluster Autoscaler.
- Karpenter can be configured to use Spot Instances, which can save a lot of money.


## What does this repo do?
- Creates an EKS Cluster with a VPC with Secondary CIDR block.
    - Secondary CIDR block is a VPC feature that allows you to add additional IP addresses to your VPC.
    - We want to use the secondary CIDR block for the pods, and the default CIDR block of the VPC for the nodes.
    - Thus defeating the IP Exhaustion problem.
- Creates 3 Private subnets in the Secondary CIDR block with `/19` mask, so we can have available IP count of `3*8190` or `24570` for our pods.
- Updates `aws-node` with Custom Networking configuration.
- Creates ENIConfig for each of our subnets in the Secondary CIDR block.
- Creates Karpenter Provisioner and AWSNodeTemplate.
- Offers troubleshooting steps for common issues.
- Recommends how to choose EC2 Instance Types. 

## Diagram

![AWS CNI and ENIConfig Diagram](https://github.com/oguzhan-yilmaz/karpenter-eks-vpc-secondary-cidr/blob/main/docs/images/secondary-cidr-block-diagram.png?raw=true)



