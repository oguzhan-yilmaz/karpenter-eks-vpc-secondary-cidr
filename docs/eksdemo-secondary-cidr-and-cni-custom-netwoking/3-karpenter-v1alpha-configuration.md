# 3. Karpenter `v1alpha` Configuration (Provider & AWSNodeTemplate)

- This configuration is for Karpenter `v1alpha` versions (or `v0.31.0` and below).
- Check out: [Migrate from AWSNodeTemplate to NodeClass](../migrate-from-awsnodetemplate-to-nodeclass.md) if you wish to upgrade `v0.32.0` and above.

### Install Karpenter

```bash
# NOTE: if you're planning to use Karpenter v1alpha and below, 
#       you should use an option to specify Karpenter version <= v0.31
#       in this eksdemo command

eksdemo install autoscaling-karpenter \
    --cluster "$CLUSTER_NAME" \
    --set "hostNetwork=true"

```

### Karpenter Configuration

- Let's delete the default karpenter config (note: important to delete them beforehand)

```bash
kubectl -n karpenter get provisioners default -o yaml
kubectl -n karpenter get awsnodetemplate default -o yaml

# delete default ones
kubectl delete provisioners default
kubectl delete awsnodetemplate default

```

#### Create Karpenter Provisioner

```bash
cat <<EOF | kubectl apply -f -
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  ttlSecondsUntilExpired: 604800 # expire nodes after 7 days (in seconds) = 7 * 60 * 60 * 24
  providerRef:
    name: default
  consolidation:
    enabled: true
  limits:
    resources:
      cpu: 1k
  requirements:
    # Include general purpose instance families
    - key: karpenter.k8s.aws/instance-family
      operator: In
      values: [c5, m5, r5]
    - key: karpenter.sh/capacity-type
      operator: In
      values:
      - on-demand
      - spot
    - key: kubernetes.io/arch
      operator: In
      values: [amd64]
  # Karpenter provides the ability to specify a few additional Kubelet args.
  # These are all optional and provide support for additional customization and use cases.
  kubeletConfiguration:
    maxPods: 20
EOF

```

#### Create Karpenter AWSNodeTemplate

-

```bash
cat <<EOF | kubectl apply -f -
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: default
spec:
  subnetSelector:
    # should be the primary cidr block subnets
    karpenter.sh/discovery: "${CLUSTER_NAME}"
  securityGroupSelector:
    karpenter.sh/discovery: "${CLUSTER_NAME}"
  amiFamily: "AL2"
  userData: |
    #!/bin/bash
    echo "Running custom user data script"
    sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
    sudo systemctl status amazon-ssm-agent
  tags:
    dev.corp.net/app: ExampleTag
    karpenter.sh/discovery: "${CLUSTER_NAME}"
    Name: "Karpenter-Node-${CLUSTER_NAME}"
  # optional, configures storage devices for the instance
  blockDeviceMappings: # AL2 machine defaults
    - deviceName: /dev/xvda
      ebs:
        volumeSize: 100Gi
        volumeType: gp3
        encrypted: true
  # optional, configures detailed monitoring for the instance
  # detailedMonitoring: "..."
EOF
```

#### Check the `.status` of the Karpenter AWSNodeTemplate

- `AWSNodeTemplate` object will update it's `.status` definition with the resolved Subnet and Security Group IDs.
- Check these IDs to make sure they are correct.

```bash
kubectl -n karpenter get awsnodetemplate default -o yaml | yq '.status'
```


### Next Steps

- [Create a load on the cluster to trigger Karpenter](create-load-on-the-cluster.md)
- [Demo Troubleshooting](troubleshooting.md)
