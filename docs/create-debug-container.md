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
```bash


```
k exec dnsutils -- nslookup kube-dns.kube-system
k exec dnsutils -- nslookup kubernetes.default
k exec dnsutils -- nslookup google.com
k exec dnsutils -- dig kubernetes.default | grep SERVER
k exec dnsutils -- dig kubernetes.default @kube-dns.kube-system | grep SERVER


k exec dnsutils -- nslookup karpenter.karpenter.svc
```