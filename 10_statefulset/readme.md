# statefulset lab

steps:

- create pv: `pv-hostpath.yaml`
- create headless service: `kubia-svc-headless.yaml`
- create statefulset: `kubia-statefulset.yaml`
- check pvc for pod: `kubectl get po kubia-0 -o yaml`
- access the pod by proxy: `kubectl proxy`

```bash
curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
# post new data 
curl -X POST -d "Hey there! This greeting was submitted to kubia-0" localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
# check data
curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
curl localhost:8001/api/v1/namespaces/default/pods/kubia-1/proxy/
```

- delete a pod: `kubectl delete po kubia-0`

```bash
# check data is still there
curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
```

# service discovery using SRV

```bash
kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local

```

# update statefulset 

1. `kubectl edit statefulset kubia`
spec.replicas:3
containers.image: luksa/kubia-pet-peers
2. if rolling udpate is not running, manually delete kubia-0,kubia-1
3. write data to service

```bash
curl -X POST -d "The sun is shining" localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
curl -X POST -d "The weather is sweet" localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
# read
curl localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
```

# simulate network partition lab

can't do this under minikube env