# Setting up Robot from scratch

## Install image and prepare robot

* Image Ubuntu 22.04, Microshift 4.8 :  [ubuntu\_2204\_gopigobits-microshift48-edgectrl-shrink.img.tgz](https://drive.google.com/file/d/1I1zhFV3aXpyn30Eg-lO2gupraSWquO7B/view?usp=drive_link)  
* Write to SD Card, will be resized to SD Card size at first boot  
  `tar xzvf ubuntu_2204_gopigobits-microshift48-edgectrl-shrink.img.tgz`  
  `sudo dd if=ubuntu_2204_gopigobits-microshift48-edgectrl-shrink.img of=/dev/sdXXX status=progress`  
    
* Attach Mini HDMI cable and keyboard  
* Login as root / r3dh4t1\!  
* Set hostname, e.g.: hostnamectl set-hostname abcwarrior.robot.lan

## Network Setup

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
          optional: true
          access-points: 
            "DO-Tech":
              password: "WSgzJN82kEMgx2Yp"
        wlan0:
          access-points:
            "redhatrobos":
              password: "r3dh4t1\!"
          dhcp4: true
```

* Or attach ethernet cable  
* Login with SSH, root is allowed  
* Install robothackathon ssh-key

## Camera Setup (Raspi camera v2)

Playbook camera-test.yaml iis here [https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot](https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot)   

* Cable orientation: blue “bar” on cable oriented to USB ports, blue bar at camera away from lens  
* Test camera is detected: vcgencmd get\_camera  
* Script to test image acquisition  

```
  import cv2  
  \# open camera  
  cap \= cv2.VideoCapture('/dev/video0', cv2.CAP\_V4L)  
    
  \# set dimensions  
  cap.set(cv2.CAP\_PROP\_FRAME\_WIDTH, 2560\)  
  cap.set(cv2.CAP\_PROP\_FRAME\_HEIGHT, 1440\)  
    
  \# take frame  
  ret, frame \= cap.read()  
  \# write frame to file  
  cv2.imwrite('/root/ramfilesystem/image.jpg', frame)  
  \# release camera  
  cap.release()
```

## Microshift

Playbook microshift-reset.yaml is here [https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot](https://github.com/cloud-native-robotz-hackathon/infrastructure/tree/main/robot) 

* To reset Microshift: systemctl stop microshift.service; rm \-rf /var/lib/microshift; systemctl start microshift.service  
* To use oc locally: export KUBECONFIG=/var/lib/microshift/resources/kubeadmin/kubeconfig  
* Or cat /var/lib/microshift/resources/kubeadmin/kubeconfig \> \~/.kube/config


## Triton

[https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting\_started/quickstart.html](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting_started/quickstart.html)

Check model  
`curl \--location \--request GET 'http://localhost:8000/v2/models/densenet\_onnx/stats'`


