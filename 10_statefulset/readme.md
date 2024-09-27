# statefulset lab

steps:

- [don't need this]create pv: `pv-hostpath.yaml`
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

- delete a pod: `kubectl delete po kubia-0`, wait for the new pod
- check data is still there: `curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/`

# service discovery using SRV

```bash
kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local
kubectl run -it srvlookup --image=jshua/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local

output:
; <<>> DiG 9.9.5-3ubuntu0.19-Ubuntu <<>> SRV kubia.default.svc.cluster.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65421
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;kubia.default.svc.cluster.local. IN    SRV

;; ANSWER SECTION:
kubia.default.svc.cluster.local. 30 IN  SRV     0 50 80 kubia-1.kubia.default.svc.cluster.local.
kubia.default.svc.cluster.local. 30 IN  SRV     0 50 80 kubia-0.kubia.default.svc.cluster.local.

;; ADDITIONAL SECTION:
kubia-0.kubia.default.svc.cluster.local. 30 IN A 10.244.0.21
kubia-1.kubia.default.svc.cluster.local. 30 IN A 10.244.0.20

;; Query time: 95 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Fri Sep 27 05:56:22 UTC 2024
;; MSG SIZE  rcvd: 350

pod "srvlookup" deleted
```

# update statefulset 

1. `kubectl edit statefulset kubia`
spec.replicas:3
containers.image: luksa/kubia-pet-peers
2. if rolling update is not running, manually delete kubia-0,kubia-1
3. create a kubia-public service:`k create -f kubia-svc-public.yaml`
4. write data to service

```bash
curl -X POST -d "The sun is shining" localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
curl -X POST -d "The weather is sweet" localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
# read
curl localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
```

# simulate network partition lab

can't do this under minikube env