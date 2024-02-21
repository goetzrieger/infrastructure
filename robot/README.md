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