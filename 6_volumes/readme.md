# use pv and pvc to store data across pods

steps:
- create pv by `kubectl create -f mongodb-pv-hostpath.yaml`
- create pvc
- create mongodb pod by `kubectl create -f mongodb-pod-pvc.yaml`
- login the mongodb and insert some data
```bash
kubectl exec -it mongodb -- mongosh
# inside mongo shell
use mystore
db.foo.insert({name:'foo'})
db.foo.find()
```
- delete the pod mongodb
- recreate the pod by `kubectl create -f mongodb-pod-pvc.yaml`
- login the mongodb and check that data is still there


