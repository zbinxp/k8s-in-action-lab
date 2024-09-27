# rolling update lab

steps:

- create v1 app and service `k create -f kubia-rs-svc-v1.yaml`
- check that the svc is running. `k get svc kubia`
`while true; do cur http://ip:nodeport; done`
- rolling update: `k rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2`
notes: pay attention to the labels of kubia-v1 service and kubia-v2 service

cons:
the rolling update is dominated by kubectl client, if the client fails during the update, the update stops in the middle state.

# deployment lab

deployment -(manage)--> replicaset --(manage)--> pods

steps:

- create a deployment, `k create -f kubia-deployment-v1.yaml --record`
- check status, `k rollout status deploy kubia`
- update image in pod template, `k set image deploy kubia nodejs=luksa/kubia:v2`

- after v2 get deployed. checkout the replicaset `k get rs`
- update to v3: `k set image deploy kubia nodejs=luksa/kubia:v3`
- as there's error, rollout `k rollout undo deploy kubia`
- rollout to specific revision: `k rollout undo deploy kubia --to-revision=1`

# how to slow down the update process

`k patch deploy kubia -p '{"spec":{"minReadySeconds":10}}'`

# deployment pause

steps:
- update to v4: `kubectl set image deploy kubia nodejs=luksa/kubia:v4`
- pause: `k rollout pause deploy kubia`
- resume: `k rollout resume depoyment kubia`

# use kubectl apply to update deployment

`k apply -f kubia-deploy-v3-with-readiness.yaml`