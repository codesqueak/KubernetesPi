# Kubernetes Raspberry Pi Cluster

This guide covers the construction, installation and use of a Kubernetes Cluster hosted on a set of Raspberry Pi computers.

One I built earlier ...
<p align="center">
  <img src="images/cluster.png">
</p>

## Hardware

A minimum configuration to demonstrate the features of Kubernetes that I used is as follows:

| Item | Use | Example | Quantity |Notes | 
|------|-----|---------|--------|---|
| Nodes | Raspberry Pi 3 Model B+   | [Amazon Link](https://www.amazon.co.uk/Raspberry-Pi-Model-64-Bit-Processor/dp/B07BDR5PDW) | 5 |Use a model with 1GB RAM and Quad Core CPU | 
| Power | Anker PowerPort 60 W 6-Port  | [Amazon Link](https://www.amazon.co.uk/Anker-PowerPort-Family-Sized-Technology-Smartphones-Black/dp/B00PK1IIJY)  |  1    |   | 
| USB Cables | Power to Pi's    | [Amazon Link](https://www.amazon.co.uk/gp/product/B00OOOHPN8) |   1 Pack       | Need 5      | 
| Switch   | Network / 8 ports |  [Amazon Link](https://www.amazon.co.uk/gp/product/B00AWM7PKO) |  1      |  Only 5 ports needed. Note 1 | 
| Ethernet Cables  | Network |  [Amazon Link](https://www.amazon.co.uk/gp/product/B01LF80T4M) | 1 Pack       | Need 5      | 
| USB to 12 Converter  | Switch Power    |    [Amazon Link](https://www.amazon.co.uk/gp/product/B071X6VYXR)    |   1     | Note 2      |
| Frame | Somewhere for your Pi's to live | [Amazon Link](https://www.amazon.co.uk/gp/product/B01G38L7VS) | 2 | |
| Storage | O/S Storage | [Amazon Link](https://www.amazon.co.uk/gp/product/B074B573C4) | 5 | Cheap and seem to work ok |

**Note 1** Netgear also produce a 5 port switch that runs from 5V. (8 port needs 12V). I wanted to run a bigger cluster so went for 8 ports.

**Note 2** The 5V to 12V cable is just to make things tidier.  You can use the original 12V PSU for the switch. Be careful to check power requirements if not using the original PSU.

## Configure Raspberry Pi Nodes

Software versions:

* O/S version: `Raspbian Stretch Lite / Kernel 4:14`
* Kubernetes version: `v1.12.2`
* Docker version: `18.06.1-ce`

Network:

* Master node: `172.16.1.200 (Host name: kubectl)`
* Worker node: `172.16.1.201..203 (Host names: worker1..3)`
* Ingress node: `172.16.1.201`
* Gateway: `172.16.1.1`
* DNS: `1.1.1.1, 8.8.8.8`

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
* Edit boot configuration 
```sudo nano  /boot/cmdline.txt``` 
...and add the following options 
```ipv6.disable=1 cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1```
* Reboot

This gives a basic configuration ready to install Docker and Kubernetes on

**_You should be able to do all steps beyond this point via SSH on the static IP address you have set_**

### Install Docker

At the time of writing, Kubernetes does not install with the latest version of Docker. To complete the install I needed to install a specific Docker version:

* Show all available versions 

```apt-cache policy docker-ce```

* Select a version and install. This is the latest that would work with my Kubernetes - YMMV

```sudo apt-get install docker-ce=18.06.1~ce~3-0~debian -qy```

* Make Docker available to pi user

```sudo usermod pi -aG docker```

### Install Kubernetes

* Install

```sudo apt-get install -qy kubeadm```

* Reboot

This is now a complete base image which can be used for control and worker nodes.  To save time, make an image copy to all of your SD cards.
Don't forget to change IP addresses and host names !

## Install Kubernetes Master Node

Select the machine you are going to use as the master node

* Load images 

```kubeadm config images pull```

### Hack Warning !

This section is a hack due to a timing issue bringing up the control plane.  Hopefully this will be [fixed](https://github.com/kubernetes/kubeadm/issues/413) at a later date...

This hack involves opening two ssh terminals, using one to modify the install as it runs.

#### First SSH Terminal

* Execute the following command
```
sudo watch -n 1.0 "sed -i 's/initialDelaySeconds: [0-9]\+/initialDelaySeconds: 360/' /etc/kubernetes/manifests/kube-apiserver.yaml"
```
This changes the `initialDelaySeconds` parameter to several minutes which gives time for everything to start.

####  Second SSH Terminal

* Execute the following command
```
sudo kubeadm init
```

* Once complete, note the line starting `kubeadm join --token`. You will need this later to join worker nodes to the cluster (Note 1)
* Stop the first SSH terminal as it is no longer needed
* Make the install generally available
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

*Note 1:* You can always regenerate the token with `kubeadm token create` at a later date


<p align="center">
  <img src="images/version.png">
</p>

## Configure Networking

Most guides use [Flannel](https://github.com/coreos/flannel) as the networking layer. However, I found it impossible to get it working so used [Weave](https://github.com/weaveworks/weave) instead.

* Install networking
```
sudo sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

* Show system to confirm install
```
kubectl get pods --all-namespaces
kubectl get nodes
``` 

## Install Kubernetes Worker Node

To add a worker node to the cluster, execute a
 
 `kubeadm join` 
 
 command using the values generated earlier.

* If the master node was at `172.16.1.200`, the command would be in the form:
```
sudo kubeadm join 172.16.1.200:6443 --token <token here>  --discovery-token-ca-cert-hash sha256:<hash value here>
```
* Add the other two worker nodes
* On the master node, execute `kubectl get nodes` to show the complete cluster

The cluster is now complete

## Publish a Basic Test Service

As a demonstration, we will now deploy the [Hypriot Busybox](https://hub.docker.com/r/hypriot/rpi-busybox-httpd/) service (A Raspberry Pi compatible Docker Image with a minimal `Busybox httpd` web server )

### Deploy the Service

* Deploy service on 2 containers
```
kubectl run hypriot --image=hypriot/rpi-busybox-httpd --replicas=2 --port=80
kubectl expose deployment hypriot --port 80
kubectl get endpoints hypriot
```
* Verify Deployment on Worker Nodes
```
curl <IP address of an endpoint (Not a worker node IP address)>

<html>
<head><title>Pi armed with Docker by Hypriot</title>
  <body style="width: 100%; background-color: black;">
    <div id="main" style="margin: 100px auto 0 auto; width: 800px;">
      <img src="pi_armed_with_docker.jpg" alt="pi armed with docker" style="width: 800px">
    </div>
  </body>
</html>
```

### Deploy a Loader Balancer and Ingress Point

Now we have the service running we need to configure a single ingress point with a load balancer.

* Install load balancer
```
kubectl apply -f https://raw.githubusercontent.com/hypriot/rpi-traefik/master/traefik-k8s-example.yaml
kubectl label node <worker node to host load balancer> nginx-controller=traefik
```
* Create an ingress point
```
cat > hypriot-ingress.yaml <<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hypriot
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: hypriot
          servicePort: 80
EOF

kubectl apply -f hypriot-ingress.yaml
```
Verify all nodes are connected and running

<p align="center">
  <img src="images/nodes.png">
</p>

Verify correct operation by pointing a browser at the ingress node:

<p align="center">
  <img src="images/hypriot.png">
</p>

**_...and this probably won't work. Recent changes in Kubernetes have introduced Role Based Access Control (RBAC)_**

### How to Bypass RBAC

For running test configurations, using [RBAC](http://docs.heptio.com/content/tutorials/rbac.html) can be painful. A (not recommended for production) way around this is:
```
kubectl create clusterrolebinding varMyClusterRoleBinding --clusterrole=cluster-admin --serviceaccount=kube-system:default
```
A reboot of the cluster will probably be needed after this.

*Note:* From a cold start, the cluster takes approximnately 5 minutes before the ingress responds with the Hypriot web page





