# AWS ec2-instance-selector CLI tool

- [AWS Docs: Instance Types](https://aws.amazon.com/ec2/instance-types/)
- Karpenter uses the price-capacity-optimized strategy. EC2 Fleet identifies the pools with the highest capacity availability for the number of instances that are launching. 
- This means that we will request Spot Instances from the pools that we believe have the lowest chance of interruption in the near term. 
- EC2 Fleet then requests Spot Instances from the lowest priced of these pools.
- Knowing this, it's important to give Karpenter the widest possible list of EC2 Instance Types.

## Installation

- [Visit Github Repo](https://github.com/aws/amazon-ec2-instance-selector)


## AWS Instance Types Summary

| EC2 Instance Type | Optimized For     |
| ----------------- | ----------------- |
| t2.micro          | General purpose   |
| m5.large          | General purpose   |
| c5.large          | Compute optimized |
| r5.large          | Memory optimized  |
| g4dn.xlarge       | GPU optimized     |

## Example CLI Commands

Export your AWS credentials.

```bash
export AWS_REGION="eu-central-1"
export AWS_PROFILE="hepapi"
```

### List as table
```bash
ec2-instance-selector \
    --memory 4 \
    --vcpus 2 \
    -o table-wide \
    --cpu-architecture x86_64/amd64
```

### Interactive mode

```bash
ec2-instance-selector \
    --memory-min 2 \
    --memory-max 8 \
    --vcpus-min 2 \
    --vcpus-max 6 \
    --gpus 0 \
    --usage-class spot \
    --network-interfaces-min 2 \
    --disk-type ssd \
    --cpu-architecture "x86_64" \
    -o interactive
```
