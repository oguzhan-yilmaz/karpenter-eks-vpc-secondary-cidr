# Migrate from AWSNodeTemplate to NodeClass (a.k.a. v1beta1 Migration)
TODO: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI  

TODO: Remember that when changing the max pods you also should pass in --kube-reserved to the kubelet extra args as otherwise it will still use the ENI limits for the default instance pod count (see #782 for more details).

TODO
    Provisioner -> NodePool
    Machine -> NodeClaim
    AWSNodeTemplate -> EC2NodeClass

coming soon...

## Plan 

Node Classes enable configuration of AWS specific settings. Each NodePool must reference an EC2NodeClass using spec.template.spec.nodeClassRef. Multiple NodePools may point to the same EC2NodeClass.

SpotENI3NodePool
    max_pods: todo calculate
SpotENI2NodePool
    max_pods: todo calculate
OnDemandENI3NodePool
    max_pods: todo calculate
OnDemandENI2NodePool
    max_pods: todo calculate


