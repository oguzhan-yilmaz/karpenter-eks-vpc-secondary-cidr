# EKS Cluster w/ Secondary CIDR and Karpenter Configuration

## Index

TODO: fix index

- [EKS Custom Networking: VPC with Secondary CIDR](eks-custom-networking-vpc-secondary-cidr.md)
- [Karpenter Configuration](karpenter-configuration-pre-v0-31.md)
- [EC2 Instance Selector](ec2-instance-selector.md)


## ENI Custom Networking Demo
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

### Demo Diagram

![AWS CNI and ENIConfig Diagram](images/secondary-cidr-block-diagram.png)


## General Notes

- 