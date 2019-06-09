# How to configurate & play poker with CHIPS
### This tuto is succesfully tested with 4 Google Cloud VPS, 1 CPU, 5GB RAM, 10 GB SSD, Ubuntu 16.04. If you run into any issue, please let me know in [CHIPS Discord](https://discordapp.com/channels/455737840169386016/455737840668770315) @PHBA2061#2530

### 1) Create 4 VM instances on Google Cloud with the specs above (or you can create just 1 instance and later clone it 3 times with a snapshot). Create Firewall rules (Ingress rules) that allow all traffics (I was too lazy to individually open each single port :joy:).
![Firewall rules](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/allow%20all%20traffic.JPG)
### 2) Clone Mylo's chips-in-a-box and follow the istructions
[chips-in-a-box repo link can be found here](https://github.com/proplatformers/chips-in-a-box). Install following this order: Chips3 => Lightning => Betpoker => Pangea . Immediately after finishing Chips3 and before installing the other 3, launch chipsd:
```
chipsd -addnode=144.217.10.241 -addnode=192.99.19.160 &
```
You can add as many nodes as you want, the nodes list can be found [here](http://chips.komodochainz.info/network).
Create chipsln folder and config file:
```
cd
mkdir .chipsln
nano config
``` 
### 3) Once you finish installing
