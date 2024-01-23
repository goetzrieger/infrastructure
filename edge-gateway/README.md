# Edge gateway


During the demo we using OpenShift Local running on the presenter laptop

**Required:**
 * Running openshift local, follow the [officia documetation](https://developers.redhat.com/products/openshift-local/overview)


Install OpenShift GitOps & OpenShift Pipelines via:
```bash
oc apply -k openshift-local/
```

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