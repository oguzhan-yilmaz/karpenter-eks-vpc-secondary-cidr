# AWS ec2-instance-selector CLI tool

https://github.com/aws/amazon-ec2-instance-selector

https://aws.amazon.com/ec2/instance-types/

## AWS Instance Types Summary

| EC2 Instance Type | Optimized For     |
| ----------------- | ----------------- |
| t2.micro          | General purpose   |
| m5.large          | General purpose   |
| c5.large          | Compute optimized |
| r5.large          | Memory optimized  |
| g4dn.xlarge       | GPU optimized     |

## Example Commands

Export your AWS credentials.

```bash
export AWS_REGION="eu-central-1"
export AWS_PROFILE="hepapi"
```

```bash
ec2-instance-selector \
    --memory 4 \
    --vcpus 2 \
    -o table-wide \
    --cpu-architecture x86_64/amd64
```

Interactive mode

```bash
ec2-instance-selector \
    --memory-min 2 \
    --memory-max 8 \
    --vcpus-min 2 \
    --vcpus-max 6 \
    --gpus 0 \
    --usage-class spot \
    --network-interfaces-min 2 \
    --disk-type ssd \n
    --cpu-architecture "x86_64" \
    -o interactive
```
