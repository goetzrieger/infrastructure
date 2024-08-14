# Local development

The goal is to improve the Workshops with just
 * A cluster at demo.redhat.com (Data Center)
 * OpenShift Local / CRC on your Laptop  (Edge gateway)
 * One Robot

## Prepare the robot

### Connect Robot to your Network (Wired)

* Connect to wired network & findout ip address
* Connect via SSH (wire) to robot
* Hostname should localhost: Set the hostname via
  ```
  echo "192.168.66.70 data.lan" | sudo tee -a /etc/hosts
  ```

### Reset MicroShift at the Robot

At the [cloud-native-robotz-hackathon/infrastructure](https://github.com/cloud-native-robotz-hackathon/infrastructure) repo:

```bash
cd automation
ansible-navigator run ./microshift-reset.yaml -l data.lan
```


## Prepare OpenShift Local / CRC (Edge-dateway)

### Start OpenShift Local

Minimal requiremntes:
 * 16 GB RAM
 * 8 CPU's
 * 120 GB Root disk

### Fetch credentials

At the [cloud-native-robotz-hackathon/infrastructure](https://github.com/cloud-native-robotz-hackathon/infrastructure) repo:

```bash
cd automation
export KUBECONFIG=$(PWD)/kubeconfig-edge-gateway

crc console --credentials
# Copy past oc admin login command
oc login -u kubeadmin -p xxx-xx-xx-xxx https://api.crc.testing:6443
```

### Deploy / Configure 

At the [cloud-native-robotz-hackathon/edge-gateway-gitops](https://github.com/cloud-native-robotz-hackathon/edge-gateway-gitops) repo:


```bash
oc apply -k bootstrap/
# Wait for all pods are ready in openshift-gitops namespace
watch oc get pods -n openshift-gitops 

# Add cluster-admin role to ArgoCD
 oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller

# Apply ArgoCD Application
oc apply -f bootstrap/application.yaml

```

Login into [Argocd WebUI](https://openshift-gitops-server-openshift-gitops.apps-crc.testing/) and sync all changes.

![ArgocD screenshot](crc-argocd.png)

## Bring Data Center & Edge together

At the [cloud-native-robotz-hackathon/infrastructure](https://github.com/cloud-native-robotz-hackathon/infrastructure) repo:


### Fetch credentials of Data Center

```bash
export KUBECONFIG=$(pwd)/kubeconfig-data-center
oc login -u admin --insecure-skip-tls-verify https://api.cluster-...

# Connect edge to data center via skupper tunnel
ansible-navigator run ./create-skupper-tunnel.yaml

# Connect robots and teams
ansible-navigator run ./update-robot-to-team.yaml -l data.lan
```

In case the ansible-navigator can not reach your openshift local, try to run the playbooks localy:

```bash
ansible-galaxy collection install kubernetes.core
ansible-galaxy collection install community.kubernetes
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.general
pip install jmespath
pip install kubernetes

ansible-playbook ./create-skupper-tunnel.yaml
ansible-playbook ./update-robot-to-team.yaml -l data.lan\
```
