# EKS Cluster w/ Secondary CIDR and Karpenter Configuration

## Index


- **Demo: AWS VPC CNI Custom Networking**
    - [About the Demo](eksdemo-secondary-cidr-and-cni-custom-netwoking/README.md)
    - [1. Create VPC with Secondary CIDR and Subnets]()
    - [2. AWS VPC CNI & ENIConfig configuration for Custom Networking](eksdemo-secondary-cidr-and-cni-custom-netwoking/2-aws-vpc-cni-configuration.md)
    - [3. Karpenter v1alpha Configuration](eksdemo-secondary-cidr-and-cni-custom-netwoking/3-karpenter-v1alpha-configuration.md)
    - [Demo Troubleshooting](eksdemo-secondary-cidr-and-cni-custom-netwoking/troubleshooting.md)
- [ec2-instance-selector CLI tool](ec2-instance-selector.md)



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

