# How to configurate & play poker with CHIPS
### This guide uses materials from:
https://docs.chips.cash/en/latest/install-ln.html

https://github.com/sg777/bet

https://www.youtube.com/channel/UCCIUvenhfwYjoKN1WJGhK6A
### This guide is succesfully tested with 4 Google Cloud VPS (upon creating new account & registering your credit/debit cards, Google Cloud will give you about $200 credits to play around with), 1 CPU, 5GB RAM, 10 GB SSD, Ubuntu 16.04. If you run into any issue, please let me know in [CHIPS Discord](https://discordapp.com/channels/455737840169386016/455737840668770315) @PHBA2061#2530

### 1) Create 4 VM instances on Google Cloud...
...4 nodes are required in case you want to run a full test setup (2 nodes are required for each dealer & 1 node is required for each player, initial setup will be the same for dealers & players, unless state otherwise) with the specs above (or you can create just 1 instance and later clone it 3 times with a snapshot). Create Firewall rules (Ingress rules) that allow all traffics (I was too lazy to individually open each single port :joy:).
![Firewall rules](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/allow%20all%20traffic.JPG)
### 2) Clone Mylo's chips-in-a-box and follow the istructions:
[chips-in-a-box repo link can be found here](https://github.com/proplatformers/chips-in-a-box). Install following this order: Chips3 => Lightning. Immediately after finishing chips3 , launch chipsd daemon:
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
save the file by pressing **CTRL+X , then Y then ENTER**, below is an example:
```
alias=phba2061
rgb=FF00FF
ipaddr=32.180.1.2
```
### 3) After installing Chips3, Lightning, Betrest & Pangea, and CHIPS finish syncing 
If installing Betrest throws these errors:
```
Makefile:27: recipe for target 'cards777.o' failed
make[1]: *** [cards777.o] Error 1
make[1]: Leaving directory '/home/phba2061/bet/privatebet'
Makefile:319: recipe for target 'build_dep1' failed
make: *** [build_dep1] Error 2
```
just ignore it for the moment, this error will be resolved later by installing libwebsockets
Launch lightningd (I usually launch it in a tmux session)
```
sudo apt-get install tmux
tmux new -s lightning
```
Then inside the tmux session you've just created
```
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
The returned IP will be used as parameter of the DCV node (this could be a public/exrernal or private/internal IP, in case of my VPS on Google Cloud, it returns an internal IP). On the same node, enter:
```
./bet dcv 10.166.0.13
```
this node is going to be the DCV.
On a 2nd node (this is going to be the BVV), enter:
```
./bet bvv 35.228.250.250
```
replace 35.228.250.250 by the public IP of your DCV node. Then on your DCV node, you should see:
```
 id:03021bcb106f96319bf0c10ed264204ca71199e0dc4d5f130d5abb60c4fc8c2c6a
CHANNELD_AWAITING_LOCKIN
CHANNELD_AWAITING_LOCKIN
CHANNELD_AWAITING_LOCKIN
```
The channel between the DCV & BVV is establishing.
![DCV&BVV](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/DCV%20lock%20in.JPG)
Open a 2nd SSH instance to your DCV and enter: 
```
lightning-cli listfunds
```
now you should see:
```
{ "outputs" :
        [{ "txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8", "output" : 1, "value" : 9499823, "status" : "confirmed" } ], 
  "channels" :
        [{ "peer_id" : "03021bcb106f96319bf0c10ed264204ca71199e0dc4d5f130d5abb60c4fc8c2c6a", "channel_sat" : 500000, "channel_total_sat" : 500000, "funding_txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8" } ] }
```
This is the proof that the channel between the DCV & BVV is establishing, wait for a moment and you should see: 
```
{ "outputs" :
        [{ "txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8", "output" : 1, "value" : 9499823, "status" : "confirmed" } ], 
  "channels" :
        [{ "peer_id" : "03021bcb106f96319bf0c10ed264204ca71199e0dc4d5f130d5abb60c4fc8c2c6a", "short_channel_id" : "3884093:1:0", "channel_sat" : 500000, "channel_total_sat" : 500000, "funding_txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8" } ] }

```
Notice the **short_channel_id** field, it means the channel has succesfully established.
On a 3rd node (this is going to be the PLAYER 1 node), enter:
```
./bet player PUBLIC_IP_OF_DCV
```
similarly you should see the **CHANNELD_AWAITING_LOCKIN** on your PLAYER1 node, and enter **lightning-cli listfunds** on your DCV will return 1 more **peer_id**, wait until you see the **short_channel_id** of the 2nd peer on your DCV node.
Then repeat the same step on the 4th node (PLAYER 2), enter:
```
./bet player PUBLIC_IP_OF_DCV
```
wait for a moment, **lightning-cli listfunds** on your DCV should return:
```
{ "outputs" :
        [{ "txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8", "output" : 1, "value" : 9499823, "status" : "confirmed" } ], 
	"channels" :
        [{ "peer_id" : "03021bcb106f96319bf0c10ed264204ca71199e0dc4d5f130d5abb60c4fc8c2c6a", "short_channel_id" : "3884093:1:0", "channel_sat" : 500000, "channel_total_sat" : 500000, "funding_txid" : "260455c6dc340cdd2ed643d6b4ea645176d7d335b97dd52f41bd45c2348882b8" },
         { "peer_id" : "03d46f6abf82ffe43ad1a34656a6dc60273ee8197c3e87a19f8a58cb23635cb810", "short_channel_id" : "3884162:1:0", "channel_sat" : 0, "channel_total_sat" : 500000, "funding_txid" : "1469f88816cc0c7ebf2dc31ba4d8ba2189d0af67b80d850c25ccaa821216ce87" },
         { "peer_id" : "0336ace765253420bc1b269f5c41ee4e8e9ac3a66b105791e18057dbcb696d15e1", "short_channel_id" : "3884192:1:0", "channel_sat" : 0, "channel_total_sat" : 500000, "funding_txid" : "f3c815429f86918bffb8ef62ae4b49e3c0645bba716584762bf7e9a16000e87a" } ] }

```
3 peers correspond to the BVV & 2 PLAYERS.
![DCV&PLAYERS](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/BVV%20lock%20in.JPG)
Now open a 2nd SSH instance on your BVV node and enter **lightning-cli listfunds**, if you see only 1 peer, then it probably stuck, **CTRL + C** to kill the bet process on all your 4 nodes, then restart it starting from the DCV => BVV => PLAYER1 => PLAYER2 , you don't have to wait this time.
This time you should see **CHANNELD_AWAITING_LOCKIN** on the BVV node, wait for a moment, the game should be ready.
![Game's ready](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20ready.JPG)
Follow the instructions the play the game, the unit is in milli-satoshi. Press **CTRL + C** to quit.
### 4) Playing with the web GUI on bet, rest_dev branch (to be update)
After establishing the channels & veryfing the cli works correctly, we can now try to play via the web GUI.
First you need to install libwebsockets:
```
cd ~
git clone https://github.com/sg777/libwebsockets.git
cd libwebsockets
mkdir build
cd build
cmake ..
make && sudo make install
sudo ldconfig /usr/local/lib
```
Switch to **rest_dev** branch then compile:
```
cd ~/bet/privatebet
git checkout rest_dev
make
```
Navigate into pangea-poker folder
```
cd ~/pangea-poker-frontend/client
git checkout poker
nano pangeapoker.js
```
Navigate to this line https://github.com/sg777/pangea-poker-frontend/blob/poker/client/pangeapoker.js#L521 by pressing **CTRL + :** then enter 521 + ENTER, replace the existing IP by your DCV node's public IP:
```
pangea.wsURI = 'ws://159.69.23.30:9000'//'ws://localhost:9000'
```
Then also replace th DCV & PLAYERS IP with your nodes's IP:
```
pangea.wsURI_bvv = 'ws://159.69.23.31:9001'
pangea.wsURI_player1 = 'ws://159.69.23.28:9002'
pangea.wsURI_player2 = 'ws://159.69.23.29:9003'
```
Then save the files.
Head back to the bet/privatebet folder, enter **./bet** on all four nodes.
```
cd ~/bet/privatebet
./bet
```
![rest_dev](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/rest_dev%20bet.JPG)
open a new SSH instance on you DCV node, launch Mylo's chips-in-a-box installer then choose **startserving**
Open a new webpage then enter
```
35.228.250.250:7777
```
replace the IP address with your DCV node public IP, you should see:
![stage1](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%201.JPG)
Click on **game** (below pangea Poker ) to initialize, you should see player1 & player2 logo pop up
![stage2](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%202.JPG)
Click on player1's seat, wait until player1's logo changes, then click on player2's seat, you should see
![stage4](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%204.JPG)
The background calculation will now take 2-3 minutes to finish. You will then see: 
![stage05](https://github.com/NOCTLJRNE/CHIPS-tuto/blob/master/img/game%20stage%205.JPG)
Now you can play the game againt yourself !
![flop](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%206%20flop.JPG)
![turn](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%207%20turn.JPG)
![river](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%208%20river.JPG)
![result](https://raw.githubusercontent.com/NOCTLJRNE/CHIPS-tuto/master/img/game%20stage%209%20result.JPG)
