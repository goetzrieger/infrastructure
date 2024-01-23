# Cloud Native Robotz Hackathon - Infrastructure

All about infrastructure provisioning for the robot hackathon.

## Overview

![Overview](overview.drawio.v2.png)

|Component (Folder)|Description|
|---|---|
|[datacenter/](datacenter/)|Main OpenShift Cluster for Training and development|
|[edge-gateway/](edge-gateway/)| Single Node OpenShift to manage all robotz.

## Networking

Network: 192.168.8.0/24
Gateway: 192.168.8.1
DNS: 192.168.8.1

|IP|Device/Name|
|---|---|
|192.168.8.1|GL-AXT1800 Router/Access-Point|
|192.168.8.2|`edge-gateway.lan,*.apps.edge-gateway.lan,api.edge-gateway.lan,api-int.edge-gateway.lan` Edge gateway / Single Node OpenShift|
|192.168.8.100-149|Robot's [inventory](https://docs.google.com/spreadsheets/d/1HIwmvbSDCddMpYYcYuls53cX771A9UED9M1vFjI9NCs/edit?usp=drive_web&ouid=117302997330820064193)
|192.168.8.150-249|DHCP Range

### WiFi

* SSID: robot-hackathon-78b09
* Wifi-Password: *Stored in red hat internal password manager*
* Router/Device-Password: *Stored in red hat internal password manager*

## Robotz inventory

[Google Sheet](https://docs.google.com/spreadsheets/d/1HIwmvbSDCddMpYYcYuls53cX771A9UED9M1vFjI9NCs/edit#gid=0)