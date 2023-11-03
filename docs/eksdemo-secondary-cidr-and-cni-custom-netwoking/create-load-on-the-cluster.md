### Autoscaling Nodes with Karpenter

```bash
# ======== CREATE A LOAD
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inflate
spec:
  replicas: 0
  selector:
    matchLabels:
      app: inflate-netshoot
  template:
    metadata:
      labels:
        app: inflate-netshoot
    spec:
      terminationGracePeriodSeconds: 0
      containers:
        - name: inflate-netshoot
          image: nicolaka/netshoot
          command: ["/bin/bash"]
          args: ["-c", "while true; do ping localhost; sleep 60;done"]
          resources:
            requests:
              cpu: 1
EOF


kubectl scale deployment inflate --replicas 4

kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter -c controller
kubectl get nodes

kubectl exec inflate-netshoot-xxxx-yyy -- nslookup kubernetes.default

kubectl exec -it inflate-netshoot-xxxx-yyy -- /bin/bash


# ======== SCALE DOWN
kubectl scale deployment inflate --replicas 0

# check karpenter logs
kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter -c controller

kubectl get nodes
```