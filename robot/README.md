# Playbooks to manage the robots

Requirements

* Laptop is connected robot-hackathon-78b09 wifi
* Robots as well
* Robot SSH key at `~/.ssh/robot-hackathon`
* Podman login registry.redhat.io


## How to execute the playbooks


```bash
$ ansible-navigator run ./camera-test.yaml
```

|Playbooks|Description|
|---|---|
|`ping-all.yaml`|Just ping all robots|
|`camera-test.yaml`|Take a camera picture of all robots an save in current directory |
|`microshift-reset.yaml`|Re init microshift - IMPORTANT: New kubeconfig is created|


## Rebuild execution environment

```bash
podman login registry.redhat.io
export VERSION=$(date +%Y%m%d%H%M)

ansible-builder build \
    --verbosity 3 \
    --container-runtime podman \
    --file execution-environment-fedora-based.yml \
    --tag quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION

podman login quay.io
podman push quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION
```
