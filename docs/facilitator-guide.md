# Facilitator guide

- [Travel kit](travel-kit.adoc)
- [Run a Hackathon](#preparing-to-run-a-hackathon)
  - [Order OpenShift Data Center env](#order-openshift-data-center-env)
  - [Setting up the environment](#setting-up-the-environment)
    - [Wifi Router](#wifi-router)
    - [🤖 Robot's](#-robots)
    - [OpenShift Edge Gateway](#openshift-edge-gateway)
    - [Assign team E-Mails to data center users](#assign-team-e-mails-to-data-center-users)
    - [Run the Hackathon](#run-the-hackathon)
- [Building Hackathon Components](#building-hackathon-components)
  - [Re-install/Update Edge-gateway](edge-gateway-applicance-builder.adoc)
  - [Setup Robot from Scratch](#setup-robot-from-scratch)
    - [Install image and prepare robot](#install-image-and-prepare-robot)
    - [Network Setup](#network-setup)
    - [Finish configuration](#finish-configuration)
    - [Camera Setup (Raspi camera v2)](#camera-setup-raspi-camera-v2)
    - [Microshift](#microshift)
    - [Triton](#triton)
  - [Setup Edge Gateway / SNO OpenShift](#setup-edge-gateway--sno-openshift)
- [Development](development.md)
    - [Local/Laptop](local-development.md)
- [Troubleshooting](troubleshooting.adoc)

# Run a Hackathon

|Check|When|What|
|---|---|---|
| |Before 1-2 Days| [Order OpenShift Data Center env](#order-openshift-data-center-env) |
| |Location|[Setting up the environment](#setting-up-the-environment)|

## Order OpenShift Data Center env

This includes deploying the hackathon environment in [demo.redhat.com](https://catalog.demo.redhat.com/catalog?search=Cloud+Native+Robot+Hackathon&item=babylon-catalog-prod%2Fsandboxes-gpte.cloud-native-robot.prod) and making sure an Edge Gateway is available.

|Field|Value
|---|---|
|OpenShift User Count|`9`|
|Region|Close as possible: `eu-central-1`|
|GPU Worker Nodes (one GPU per Worker!)|For testing puropose only, workshop is not prepred to use GPU's!
|Enable workshop user interface|True|

## Setting up the environment

### Wifi Router

* Start the Wifi router and attach to the local Wifi or wire
  * SSID: `robot-hackathon-78b09`
  * Wifi-Password: Stored in RH Bitwarden collection
  * The router is a preconfigured GL.iNet AXT1800, the configuration to restore is here (always use latest!): [gDrive router backup](https://drive.google.com/drive/folders/19ZIPrv9bnL4JvYXGgUOYihp5AsKfzZPa?usp=drive_link) (RH internal only)
  * Check connectivity to Internet

### 🤖 Robot's

* Unpack all Robots
* Attach robots to power and start up
* Boot all Robots.
* Wait a couple of minutes...
* Connect your Laptop to Wifi `robot-hackathon-78b09`
* Check connection via ansile

  <details>
  <summary>Details</summary>

  At the infrastructure repo:

  ```bash
  % ansible-navigator run ./ping-all.yaml
  ...
  ```

  And let it dance via:

  ```bash
  % ansible-navigator run ./move-robots.yaml
  ...
  ```
  </details>

### OpenShift Edge Gateway

The Edge Gateway is a OCP SNO instance running on-premise in the same Wifi network as the robots. All connections from the hackathon OCP cluster to the robots go through it.

It is usually prepared and ready to use. Just connected the network wire to the router.

* Power on after Wifi is available
* Login into [OpenShift Console](https://console-openshift-console.apps.edge-gateway.lan/) and check all operators
  * User: kubeadmin
  * Password: *stored at bitwarden*
* Login into [OpenShift GitOps](https://openshift-gitops-server-openshift-gitops.apps.edge-gateway.lan/) and check everything is in synced
* Connect edge-date with data center

  <details>
  <summary>Run playbooks...</summary>

  At the [cloud-native-robotz-hackathon/infrastructure](https://github.com/cloud-native-robotz-hackathon/infrastructure) repo:

  ```bash
  cd infrastructure/automation/

  # Login into data center
  export KUBECONFIG=kubeconfig-data-center
  oc login -u admin --insecure-skip-tls-verify https://api.cluster-...

  # Login into edge gateway
  export KUBECONFIG=kubeconfig-edge-gateway
  oc login -u admin --insecure-skip-tls-verify https://edge-gateway.lan:6443

  # Optional: Adjust robot to team configuration
  vim inventory.yaml

  # Run playbook
  ansible-navigator run new-data-center.yaml  -l <select location / specific robot>
  ```

  Tip: If the playbook fails, this is propably due to a [bug](https://github.com/cloud-native-robotz-hackathon/infrastructure/issues/66) where the Interconnect Controller doesn't initalize correctly. You can restart the Interconnect Pod (skupper-site-controller-xxx...) in the openshift-operators project as a workaround. Once done, rerun the Ansible playbook.

  </details>

## Assign team E-Mails to data center users

|E-Mail|User|
|---|---|
|team-1@example.com|team-1|
|team-2@example.com|team-2|
|team-3@example.com|team-3|
|team-4@example.com|team-4|
|team-5@example.com|team-5|
|team-6@example.com|team-6|
|team-7@example.com|team-7|
|team-8@example.com|team-8|
|team-9@example.com|team-9|

## Run the Hackathon

Have fun!

# Building Hackathon Components

The steps detailed here should only be done when rebuilding hackathon components which are normally ready to use.

## Setup Robot from Scratch

This should only be neccessary with a new robot or when repairing/updating/replacing a robot.

### Install image and prepare robot

* Image Ubuntu 22.04, Microshift 4.8 :  [ubuntu_2204_gopigobits-microshift48-edgectrl-shrink.img.tgz](https://drive.google.com/file/d/1I1zhFV3aXpyn30Eg-lO2gupraSWquO7B/view?usp=drive_link)
* Write to SD Card, will be resized to SD Card size at first boot
  `tar xzvf ubuntu_2204_gopigobits-microshift48-edgectrl-shrink.img.tgz`
  `sudo dd if=ubuntu_2204_gopigobits-microshift48-edgectrl-shrink.img of=/dev/sdXXX status=progress`

* Attach network cable
* Login as root / password
* CHECK IF DONE IN IMAGE ALREADY: Configure Wifi

### Network Setup

* For WLAN edit /etc/netplan/50-cloud-init.yaml and add your WLAN access point, run netplan apply. Config example:

```
  network:
      ethernets:
          eth0:
              dhcp4: true
              optional: true
      version: 2
      Wifis:
        wlan0:
          access-points:
            "redhatrobos":
              password: "password"
          dhcp4: true
```
### Finish configuration

Run Playbook `automation/configure-robot.yaml` against new robot to finish setup.

### Camera Setup (Raspi camera v2)

Playbook camera-test.yaml is here [https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot](https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot)

* Cable orientation: blue “bar” on cable oriented to USB ports, blue bar at camera away from lens
* Test camera is detected: vcgencmd get_camera
* Script to test image acquisition

```
  import cv2
  # open camera
  cap = cv2.VideoCapture('/dev/video0', cv2.CAP_V4L)

  # set dimensions
  cap.set(cv2.CAP_PROP_FRAME_WIDTH, 2560)
  cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 1440)

  # take frame
  ret, frame = cap.read()
  # write frame to file
  cv2.imwrite('/root/ramfilesystem/image.jpg', frame)
  # release camera
  cap.release()
```

### Microshift

Playbook microshift-reset.yaml is here [https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot](https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot)

* To reset Microshift: systemctl stop microshift.service; rm -rf /var/lib/microshift; systemctl start microshift.service
* To use oc locally: export KUBECONFIG=/var/lib/microshift/resources/kubeadmin/kubeconfig
* Or cat /var/lib/microshift/resources/kubeadmin/kubeconfig > ~/.kube/config

### Triton

[https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting\_started/quickstart.html](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting_started/quickstart.html)

Check model
`curl --location --request GET 'http://localhost:8000/v2/models/densenet_onnx/stats'`

## Setup Edge Gateway / SNO OpenShift
