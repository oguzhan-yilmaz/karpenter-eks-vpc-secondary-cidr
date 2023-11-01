# Troubleshooting Guide
- TODO: debug container dns utils
- TODO: aws cli ssh command
- TODO: karpenter, coredns, aws-node log commands


## Helpful bash functions
```bash
alias klogs_karpenter="kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter"
alias klogs_coredns="kubectl logs -f -n kube-system deploy/coredns"
alias klogs_aws_node="kubectl logs -f -n kube-system -l k8s-app=aws-node"




```

### AWS CLI SSM Session Manager

- [Install AWS CLI SSM Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
- EC2 instance must have SSM Agent installed (possibly in userdata)
- Connect to EC2 instance via SSM Session Manager, or you can use the AWS Console UI
    ```bash
    # you can SSH into the Karpenter nodes like this
    aws ssm start-session --target i-061f1a56dfff5d8f3
    ```

#### Error: Address is not allowed

- You can get the following error if you forget to set `hostNetwork: true` in the karpenter deployment.
```bash
Error from server (InternalError): error when creating "STDIN": Internal error occurred: failed calling webhook "defaulting.webhook.karpenter.k8s.aws": failed to call webhook: Post "https://karpenter.karpenter.svc:8443/default/karpenter.k8s.aws?timeout=10s": Address is not allowed
```


### EKS Health Issues
TODO: here
- You may get this error if you route the Subnets on Secondary CIDR to a Internet Gateway, making it a public subnet.
- If this is the case, you must enable auto-assign public IP address for the subnet.  

```bash
Ec2SubnetInvalidConfiguration
	One or more Amazon EC2 Subnets of [subnet-00782ed1060ae5f88, subnet-0af9794264f7165bc, subnet-0b974d5872910ab7b] for node group mymymy does not automatically assign public IP addresses to instances launched into it. If you want your instances to be assigned a public IP address, then you need to enable auto-assign public IP address for the subnet. See IP addressing in VPC guide: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html#subnet-public-ip
```


### Test Pods having IP addresses from Secondary CIDR Block
```bash
kubectl create deployment nginx --image=nginx
kubectl scale --replicas=3 deployments/nginx
kubectl expose deployment/nginx --type=NodePort --port 80


kubectl port-forward svc/nginx 8080:80 &

curl -Lk localhost:8080

bg # and then+ Ctrl-C

# try to see if the pods are running on the secondary CIDR block (ignore daemonset pods)
kubectl get pods -o wide
```


### 
```bash
```