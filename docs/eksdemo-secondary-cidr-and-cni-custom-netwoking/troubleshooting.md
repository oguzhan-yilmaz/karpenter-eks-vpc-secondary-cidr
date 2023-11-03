# Troubleshooting Guide

### Helpful bash functions

```bash
alias klogs_karpenter="kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter"
alias klogs_coredns="kubectl logs -f -n kube-system deploy/coredns"
alias klogs_aws_node="kubectl logs -f -n kube-system -l k8s-app=aws-node"
```

### Debug Cluster DNS

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "infinity"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
  # nodeName: ip-105-64-46-249.eu-central-1.compute.internal
EOF

# try to resolve cluster DNS from the pod
kubectl exec dnsutils -- nslookup kube-dns.kube-system
kubectl exec dnsutils -- nslookup kubernetes.default
kubectl exec dnsutils -- nslookup google.com

# if karpenter is installed
kubectl exec dnsutils -- nslookup karpenter.karpenter.svc
```

### Test Pods having IP addresses from Secondary CIDR Block

```bash
kubectl create deployment nginx --image=nginx
kubectl scale --replicas=3 deployments/nginx
kubectl expose deployment/nginx --type=NodePort --port 80


kubectl port-forward svc/nginx 9090:80
# check localhost:9090 on browser

# try to see if the pods are running on the secondary CIDR block
# (p.s. ignore daemonset pods or hostNetwork:true pods)
kubectl get pods -o wide
```

### AWS CLI SSM Session Manager

- [Install AWS CLI SSM Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
- EC2 instance must have SSM Agent installed (possibly in userdata)
- Connect to EC2 instance via SSM Session Manager, or you can use the AWS Console UI.
  ```bash
  # you can SSH into the Karpenter nodes like this
  aws ssm start-session --target i-061f1a56dfff5d8f3
  ```

### Error: Address is not allowed

- You can get the following error if you forget to set `hostNetwork: true` in the Karpenter deployment.

```bash
Error from server (InternalError): error when creating "STDIN": Internal error occurred: failed calling webhook "defaulting.webhook.karpenter.k8s.aws": failed to call webhook: Post "https://karpenter.karpenter.svc:8443/default/karpenter.k8s.aws?timeout=10s": Address is not allowed
```

### EKS Health Issues

- You may get this error if you route the Subnets on Secondary CIDR to a Internet Gateway, making it a public subnet.
- If this is the case, you must enable auto-assign public IP address for the subnet.

```bash
Ec2SubnetInvalidConfiguration
	One or more Amazon EC2 Subnets of [subnet-00782ed1060ae5f88, subnet-0af9794264f7165bc, subnet-0b974d5872910ab7b] for node group mymymy does not automatically assign public IP addresses to instances launched into it. If you want your instances to be assigned a public IP address, then you need to enable auto-assign public IP address for the subnet. See IP addressing in VPC guide: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html#subnet-public-ip
```
