- [Preparing to Run a Hackathon](#preparing-to-run-a-hackathon)
  - [Setup OpenShift Environment](#setup-openshift-environment)
    - [Configure \& Deploy RHDP Catalog Item "Cloud Native Robot Hackathon"](#configure--deploy-rhdp-catalog-item-cloud-native-robot-hackathon)
    - [OpenShift Edge Gateway](#openshift-edge-gateway)
  - [Hackathon On-Premise Setup](#hackathon-on-premise-setup)
    - [WIFI Router](#wifi-router)
    - [Edge Gateway](#edge-gateway)
    - [Robots](#robots)
    - [Connect RHDP Hackathon Cluster \& Edge Gateway](#connect-rhdp-hackathon-cluster--edge-gateway)
      - [Fetch credentials of RHDP Cluster](#fetch-credentials-of-rhdp-cluster)
      - [Connect edge to data center via skupper tunnel](#connect-edge-to-data-center-via-skupper-tunnel)
- [Building Hackathon Components](#building-hackathon-components)
  - [Setup Robot from Scratch](#setup-robot-from-scratch)
    - [Install image and prepare robot](#install-image-and-prepare-robot)
    - [Network Setup](#network-setup)
    - [Finish configuration](#finish-configuration)
    - [Camera Setup (Raspi camera v2)](#camera-setup-raspi-camera-v2)
    - [Microshift](#microshift)
    - [Triton](#triton)
  - [Setup Edge Gateway / SNO OpenShift](#setup-edge-gateway--sno-openshift)
- [Troubleshooting](#troubleshooting)


# Preparing to Run a Hackathon 

## Setup OpenShift Environment

This includes deploying the hackathon environment in RHDP and making sure an Edge Gateway is available.

### Configure & Deploy RHDP Catalog Item "Cloud Native Robot Hackathon"

-> Sizing

### OpenShift Edge Gateway

The Edge Gateway is a OCP SNO instance running on-premise in the same WIFI network as the robots. All connections from the hackathon OCP cluster to the robots go through it.

It is usually prepared and ready to use.

## Hackathon On-Premise Setup

### WIFI Router

* Start the WIFI router and attach to the local WIFI
  * SSID: robot-hackathon-78b09 Wifi-Password: Stored in RH Bitwarden collection
  * The router is a preconfigured GL.iNet AXT1800, the configuration to restore is here (always use latest!): [https://drive.google.com/file/d/14T-on2pO0IIXGgDSKp70ZaBU8B9nxKvV/view?usp=drive_link](https://drive.google.com/drive/folders/19ZIPrv9bnL4JvYXGgUOYihp5AsKfzZPa?usp=drive_link) (RH internal only)
  * Check connectivity to Internet

### Edge Gateway

* Power on after WIFI is available
* -> How to check?

### Robots

* Attach robots to power and start up
 * Wait a couple of minutes and run automation/move-robots.yaml Playbook from laptop attached to hackathon WIFI.
 * All robots should do the dance!

### Connect RHDP Hackathon Cluster & Edge Gateway

At the [cloud-native-robotz-hackathon/infrastructure](https://github.com/cloud-native-robotz-hackathon/infrastructure) repo:

#### Fetch credentials of RHDP Cluster

```bash
cd automation
export KUBECONFIG=$(pwd)/kubeconfig-data-center
oc login -u admin --insecure-skip-tls-verify https://api.cluster-...
```

#### Connect edge to data center via skupper tunnel
```
ansible-navigator run ./create-skupper-tunnel.yaml
```

If the playbook fails, this is propably due to a [bug](https://github.com/cloud-native-robotz-hackathon/infrastructure/issues/66) where the Interconnect Controller doesn't initalize correctly. You can restart the Interconnect Pod (skupper-site-controller-xxx...) in the openshift-operators project as a workaround. Once done, rerun the Ansible playbook.
   
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
* CHECK IF DONE IN IMAGE ALREADY: Configure WIFI

### Network Setup

* For WLAN edit /etc/netplan/50-cloud-init.yaml and add your WLAN access point, run netplan apply. Config example:

```
  network:
      ethernets:
          eth0:
              dhcp4: true
              optional: true
      version: 2
      wifis:
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

# Troubleshooting