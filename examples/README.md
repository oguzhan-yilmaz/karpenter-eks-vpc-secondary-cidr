# Karpenter v1beta Configuration Examples


## Index
- 

https://karpenter.sh/docs/upgrading/v1beta1-migration/
```bash
 kubectl apply -f https://raw.githubusercontent.com/aws/karpenter/v0.32.1/pkg/apis/crds/karpenter.sh_nodepools.yaml
 kubectl apply -f https://raw.githubusercontent.com/aws/karpenter/v0.32.1/pkg/apis/crds/karpenter.sh_nodeclaims.yaml
 kubectl apply -f https://raw.githubusercontent.com/aws/karpenter/v0.32.1/pkg/apis/crds/karpenter.k8s.aws_ec2nodeclasses.yaml



envsubst < examples/v1beta/node-pool.yaml | kubectl apply -f -
envsubst < examples/v1beta/ec2-node-class.yaml | kubectl apply -f -



```


