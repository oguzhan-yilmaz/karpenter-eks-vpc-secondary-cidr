# karpenter-eks-vpc-secondary-cidr
Docs on creating an EKS setup with Secondary CIDR block for IP addresses.


## Requirements
- eksdemo
- aws cli 
- jq
- yq

## Index



### General Notes

- Karpenter uses the price-capacity-optimized strategy. EC2 Fleet identifies the pools with the highest capacity availability for the number of instances that are launching. This means that we will request Spot Instances from the pools that we believe have the lowest chance of interruption in the near term. EC2 Fleet then requests Spot Instances from the lowest priced of these pools.