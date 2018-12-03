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


## Install Kubernetes Control Node



## Install Kubernetes Worker Node



## Run an Application








