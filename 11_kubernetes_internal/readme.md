# leader eletion sidecar lab

## sidecar test

```bash
kubectl run leader-elector --image=k8s.gcr.io/leader-elector:0.5 --replicas=3 -- --election=example --http=0.0.0.0:4040
kubectl proxy

curl http://localhost:8001/api/v1/namespaces/default/pods/<leader-pod-name>:4040/proxy
```

## nodejs app with election sidecar

steps:

- start the deployment: `k create -f election_sidecar_rs.yaml`
- check out: `kubectl exec elector -c nodejs-app -- wget -qO- http://localhost:8080`
- stop the leader pod: `k delete po <leader-pod>`
- check out again

[ref(https://github.com/kubernetes-retired/contrib/blob/master/election/README.md)]

# how does a service be implemented in k8s

iptable proxy mode:

- when a service get created, kube-proxy on each node will get notified via API server about the service. 
- `kube-proxy` then update the iptables on the node to reflect the service spec.
- when a request from a pod is sent, iptables on the node will udpate the packet and distribute the packet to appropriate destination
