# How to configurate & play poker with CHIPS
### This tuto is succesfully tested with 4 Google Cloud VPS, 1 CPU, 5GB RAM, 10 GB SSD, Ubuntu 16.04. If you run into any issue, please let me know in [CHIPS Discord](https://discordapp.com/channels/455737840169386016/455737840668770315) @PHBA2061#2530

### 1) Create 4 VM instances on Google Cloud...
...with the specs above (or you can create just 1 instance and later clone it 3 times with a snapshot). Create Firewall rules (Ingress rules) that allow all traffics (I was too lazy to individually open each single port :joy:).
![Firewall rules](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/allow%20all%20traffic.JPG)
### 2) Clone Mylo's chips-in-a-box and follow the istructions:
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
Enter these lines in the config file:
```
alias=<your unique alias name, visible on ln explorer>
rgb=<RGB color of your node on the ln explorer>
ipaddr=<your public ip address>
```
save the file, below is an example:
```
alias=phba2061
rgb=FF00FF
ipaddr=32.180.1.2
```
### 3) After installing Chips3, Lightning, Betpoker & Pangea, and CHIPS finish syncing 
launch lightningd (I usually launch it in a tmux session)
```
sudo apt-get install tmux
tmux new -s lightning
lightningd --log-level=debug &
```
CTRL + B, then D to detach from the tmux session.
to get your lightning p2sh address
```
lightning-cli dev-listaddrs
```
returned result
```
{ "addresses" : 
	[ 
		{ "keyidx" : 0, "pubkey" : "031e22b03a269ca681331f3fb1ce885896728ff536fadc4418b1892f63dcb708e2",
		"p2sh" : "bYwk3PgQ2nvoSpb8rVjXPg1J93fG9Br5rr", 
		"p2sh_redeemscript" : "0014d7c85c95b954971c64e33e460fc57a7ff4f5cad7",
		"bech32" : "chips1q6ly9e9de2jt3ce8r8erql3t60l60tjkhl92l0k", 
		"bech32_redeemscript" : "d7c85c95b954971c64e33e460fc57a7ff4f5cad7" } ] }
```
in the example above, **bYwk3PgQ2nvoSpb8rVjXPg1J93fG9Br5rr** is your p2sh deposit address, send 0.1 CHIPS to all 4 p2sh addresses oon 4 nodes.
