# EKS Cluster w/ Secondary CIDR and Karpenter Configuration

## Index

TODO: fix index
- [EKS Custom Networking: VPC with Secondary CIDR](eks-custom-networking-vpc-secondary-cidr.md)
- [Karpenter Configuration](karpenter-configuration-pre-v0-31.md)
- [EC2 Instance Selector](ec2-instance-selector.md)

## Why this is needed?

- Running many nodes in EKS can cause IP address exhaustion in the VPC.
- How many IP addresses are available to a node is determined by nodes ENI capacity.
    - Because of this, EKS requires running many nodes to keep up with the Pod count.
- Using a VPC with Secondary CIDR block allows us to have more IP addresses available to our pods.
- Karpenter is a faster option for cluster autoscaling than the default EKS Cluster Autoscaler.
- Karpenter can be configured to use Spot Instances, which can save a lot of money.


## What does this repo do?
- Creates an EKS Cluster with a VPC with Secondary CIDR block.
    - Secondary CIDR block is a VPC feature that allows you to add additional IP addresses to your VPC.
- Creates 3 Private subnets in the Secondary CIDR block with `/19` mask, so we can have available IP count of `3*8190` or `24570` for our pods.
- Updates `aws-node` with Custom Networking configuration.
- Creates ENIConfig for each of our subnets in the Secondary CIDR block.
- Creates Karpenter Provisioner and AWSNodeTemplate.
- Offers troubleshooting steps for common issues.
- Recommends how to choose EC2 Instance Types. 

## Diagram

![AWS CNI and ENIConfig Diagram](images/secondary-cidr-block-diagram.png)


## General Notes

- 