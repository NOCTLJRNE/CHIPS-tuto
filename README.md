# How to configurate & play poker with CHIPS
### This tuto is succesfully tested with 4 Google Cloud VPS, 1 CPU, 5GB RAM, 10 GB SSD, Ubuntu 16.04. If you run into any issue, please let me know in [CHIPS Discord](https://discordapp.com/channels/455737840169386016/455737840668770315) @PHBA2061#2530

### 1) Create 4 VM instances on Google Cloud...
...with the specs above (or you can create just 1 instance and later clone it 3 times with a snapshot). Create Firewall rules (Ingress rules) that allow all traffics (I was too lazy to individually open each single port :joy:).
![Firewall rules](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/allow%20all%20traffic.JPG)
### 2) Clone Mylo's chips-in-a-box and follow the istructions:
[chips-in-a-box repo link can be found here](https://github.com/proplatformers/chips-in-a-box). Install following this order: Chips3 => Lightning => Betrest => Pangea . Immediately after finishing Chips3 and before installing the other 3, launch chipsd:
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
### 3) After installing Chips3, Lightning, Betrest & Pangea, and CHIPS finish syncing 
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
in the example above, **bYwk3PgQ2nvoSpb8rVjXPg1J93fG9Br5rr** is your p2sh deposit address, send 0.1 CHIPS to all 4 p2sh addresses on 4 nodes (at the moment of writing this guide, Agama wallet was compromised, so you can only use either chips-cli or chips-qt to manage your funds).
```
chips-cli sendtoaddress "bYwk3PgQ2nvoSpb8rVjXPg1J93fG9Br5rr" 0.1
```
Wait for the transactions to be confirmed:
```
lightning-cli listfunds
```
returned result:
```
{ "outputs" : 
	[ 
		{ "txid" : "f9aeeaee05aff0ae9600dd2bb7e0fc2bacb2f0dc6a34208470082fe7198dcb6e", "output" : 0, "value" : 10000000, "status" : "confirmed" } ], 
		"channels" : [  ] }
```
then, on all 4 nodes:
```
cd ~
cd bet/privatebet
git checkout poker
make
```
We'll need to establish the channels between our nodes, the easiest way to do this is play on the bet/poker branch first and let the back-end handle the establishment.
On 1 of your node, enter:
```
hostname -I
```
returned result:
```
10.166.0.13
```
The returned IP  will be used as parameter of the DCV node. On the same node, enter:
```
./bet dcv 10.166.0.13
```
this node is going to be the DCV.
On a 2nd node (this is going to be the BVV), enter:
````
./bet bvv 35.228.250.250
```
35.228.250.250 is the public IP of your DCV node, on your DCV node, you should see:
```
 id:03021bcb106f96319bf0c10ed264204ca71199e0dc4d5f130d5abb60c4fc8c2c6a
CHANNELD_AWAITING_LOCKIN
CHANNELD_AWAITING_LOCKIN
CHANNELD_AWAITING_LOCKIN
```
The channel between the DCV & BVV is establishing.
Open a 2nd SSH instance to your DCV and enter: 
```
lightning-cli listfunds
```
now you should see:
