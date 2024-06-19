# Edge gateway

Login into <https://console-openshift-console.apps.edge-gateway.lan/>

## Bootstrap

Please take a look into <https://github.com/cloud-native-robotz-hackathon/edge-gateway-gitops>


## hub-controller test commands:


```
export ROBOT=data

curl -X POST -v -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/forward/10?user_key=$ROBOT
curl -X POST -v -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/backward/10?user_key=$ROBOT

curl -X POST -v -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/left/10?user_key=$ROBOT


curl -X POST -v -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/right/10?user_key=$ROBOT


curl -X GET -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/remote_status?user_key=data

curl -X GET -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/power?user_key=data

curl -X GET -k https://web-hub-controller.apps.edge-gateway.lan/api/robot/distance?user_key=data

```

## ToDo - rewrite

Added cluster to argocd instance

```bash
# Login into Robot
oc login -u kubeadmin ....

# Login into OpenShift GitOps at OpenShift Local instance
argocd login ...

# Add cluster to argocd instance
argocd cluster add $(oc config current-context )
```

Example outpur of `argocd cluster list`:
```bash
$ argocd cluster list
SERVER                          NAME         VERSION  STATUS   MESSAGE                                              PROJECT
https://10.89.114.152:6443      goetz-robot           Unknown  Cluster has no application and not being monitored.
https://kubernetes.default.svc  in-cluster            Unknown  Cluster has no application and not being monitored.
```

Apply ArgoCD Application:
```bash
oc apply -f openshift-local/starter-app.application.yaml
oc apply -f openshift-local/hubcontroller-rest.application.yaml
```
## hubcontroller-rest test commands

```bash
# Forward
curl -k -X POST  https://test-hubcontroller-rest.apps.edgesno.example.com/api/robot/forward/1?user_key=terminator

```