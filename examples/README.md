# Karpenter v1beta Configuration Examples


## Index
- 


```bash
export CLUSTER_NAME="bambi" 


cd examples/v1beta
envsubst < provisioner/spot.yaml

envsubst < examples/v1beta/node-pool.yaml | tee | kubectl apply -f -
envsubst < examples/v1beta/ec2-node-class.yaml | tee | kubectl apply -f -



```


