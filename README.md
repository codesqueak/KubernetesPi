# Kubernetes Raspberry Pi Cluster

This guide covers the construction, installation and use of a Kubernetes Cluster hosted on a set of Raspbeey Pi computers.

One I build earlier ...
<p align="center">
  <img src="images/cluster.png">
</p>


## Hardware

A minimum configuration to demonstrate the features of Kubernetes that I used is as follows:

| Item | Use | Example | Quantity |Notes | 
|------|-----|---------|--------|---|
| Nodes | Raspberry Pi 3 Model B+   | [Amazon Link](https://www.amazon.co.uk/Raspberry-Pi-Model-64-Bit-Processor/dp/B07BDR5PDW) | 4 |Use a model with 1GB RAM and Quad Core CPU | 
| Power | Anker PowerPort 60 W 6-Port  | [Amazon Link](https://www.amazon.co.uk/Anker-PowerPort-Family-Sized-Technology-Smartphones-Black/dp/B00PK1IIJY)  |  1    |   | 
| USB Cables | Power to Pi's    | [Amazon Link](https://www.amazon.co.uk/gp/product/B00OOOHPN8) |   1 Pack       | Need 4      | 
| Switch   | Network / 8 ports |  [Amazon Link](https://www.amazon.co.uk/gp/product/B00AWM7PKO) |  1      |  Only 5 ports needed. Note 1 | 
| Ethernet Cables  | Network |  [Amazon Link](https://www.amazon.co.uk/gp/product/B01LF80T4M) | 1 Pack       | Need 4      | 
| USB to 12 Converter  | Switch Power    |    [Amazon Link](https://www.amazon.co.uk/gp/product/B071X6VYXR)    |   1     | Note 2      |
| Frame | Somewhere for your Pi's to live | [Amazon Link](https://www.amazon.co.uk/gp/product/B01G38L7VS) | 2 | |
| Storage | O/S Storage | [Amazon Link](https://www.amazon.co.uk/gp/product/B074B573C4) | 4 | Cheap and seem to work ok |

**Note 1** Netgear also do a 5 port switch that runs from 5V. (8 port needs 12V). I wanted to run a bigger cluster so went for 8 ports.

**Note 2** The 5V to 12V cable is just to make things tidier.  You can use the original 12V PSU for the switch. Be careful to check power requirements if not using the original PSU.


## Configure Raspberry Pi Nodes 

### Install O/S

* Install Raspbian Stretch Lite [Download](https://www.raspberrypi.org/downloads/raspbian/) onto sn SD card and boot
* Run the config tool (`sudo raspi-config`) and set the `hostname` and enable `ssh`
* Update networking to set a static IP, Gateway and DNS addresses (IPV4 only for now). `sudo nano /etc/dhcpcd.conf`
* Disable the swap file
```
sudo dphys-swapfile swapoff
sudo dphys-swapfile uninstall
sudo systemctl disable dphys-swapfile
```
* Add Docker repository
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo "deb [arch=armhf] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```
* Add Kubernetes repository
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
```
* Update software
```
sudo apt update -qy
sudo apt upgrade -qy
```
* Edit boot configuration `sudo nano  /boot/cmdline.txt` and add the following options 
```ipv6.disable=1 cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1```
* Reboot

This gives a basic configuration ready to install Docker and Kubernetes on

### Install Docker

At the time of writing, Kubernetes does not install with the latest version of Docker. To complete the install I needed to install a specific Docker version:

* Show all available versions 
```apt-cache policy docker-ce```
* Select a version and install. This is the latest that would work with my Kubernetes - YMMV
```sudo apt-get install docker-ce=18.06.1~ce~3-0~debian -qy```
* Make Docker available to pi user
```sudo usermod pi -aG docker```

### Install Kubernetes

```sudo apt-get install -qy kubeadm```

This is now a complete base image which can be used for control and worker nodes.  To save time, make an image copy to all of your SD cards.
Don't forget to change IP addresses and node host names !

## Install Kubernetes Control Node



## Install Kubernetes Worker Node



## Run an Application








