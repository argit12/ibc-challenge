# ibc-challenge

# ibc-transactions-for-kichain
## How to perform IBC transactions in Kichain

This guide describes how to perform IBC transactions between Kichain and Rizon networks. 
We will be setting up the repeater on Rizon node.
Firstly you need to check are IBC-transfers are enabled.
```
rizond q ibc-transfer params
```
The output should provide:
```
receive_enabled: true 
send_enabled: true  
```
Download and install Relayer v1.0.0 from official source
```
git clone https://github.com/cosmos/relayer.git
cd relayer
make install
cd
rly version
```
Correct output is version: 1.0.0-rc1–152-g112205b

Initialize the repeater
```
rly config init
```
Make a relayer config directory
```
mkdir rly_config
cd rly_config
```
Now we will create a Kichain config file using a global rpc
```
nano kichain-t-3.json
```
Insert these settings in the file and save it
```
{  "chain-id": "kichain-t-3",   "rpc-addr": "https://rpc-challenge.blockchain.ki:443",    "account-prefix": "tki",   "gas-adjustment": 1.5,   "gas-prices": "0.025utki",   "trusting-period": "48h" }  
```
Also we need to create a file for Rizon chain
```
nano groot-011.json
```
Insert these settings in the file and save it
```
{  "chain-id": "groot-011",  "rpc-addr": "http://localhost:26657",   "account-prefix": "rizon",  "gas-adjustment": 1.5,  "gas-prices": "0.025uatolo",  "trusting-period": "48h" }
```
Parse config files
```
rly chains add -f groot-011.json
rly chains add -f kichain-t-3.json
cd
```
Create wallets for each chain
```
rly keys add groot-011 <name of your wallet> 
rly keys add kichain-t-3 <ame of your wallet>
```
Parse wallet files
```
rly chains edit groot-011 key name of your wallet
rly chains edit kichain-t-3 key name of your wallet
```
Change standart timeout
```
nano ~/.relayer/config/config.yaml
timeout: 30s  
```
Next, you will need to get some coins in each wallet. After that, check the availability of funds
```
rly q balance groot-011
rly q balance kichain-t-3
```
Initialize light clients for both networks
```
rly light init groot-011 -f
rly light init kichain-t-3 -f
```
Generate a path between networks 
```
rly paths generate groot-011 kichain-t-3 transfer  --port=transfer
```
Open a channel for relaying 
```
rly tx link transfer  --debug
```
If channel doesn't open, try erasing lines containting
```
client-id: 07-tendermint-16  connection-id: connection-14  channel-id: channel-11
```
in nano /root/.relayer/config/config.yaml

Re-initialize clients
```
rly light init groot-011 -f
rly light init kichain-t-3 -f
```
Open a channel
```
rly tx link transfer  --debug
```
Now by calling this command you will be able to see that we are ready to send transactions
```
rly paths list -d
```
You can perform transactions with the command
```
rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags] 
```
In my case command should look like following
```
rly tx transfer groot-011 kichain-t-3 1000000uatolo tki1fza3s64p5uvtusqlqecyxanvlusy9x9h9vqu57 --path transfer
```
We are transfering tokens from Rizon to Kichain
  
Now we can see our transaction in explorer
https://dev.mintscan.io/rizon/txs/634104F5AC84FDD8446903C93F7E8E8A48429433697364968FD4C06476DA98EA
https://dev.mintscan.io/rizon/txs/341DF3CA9DBE12E4BD557B8A29581F89FB4B4DAD2C0FC60160577DAB12F9D709

Let's try to send tokens from Kichain to Rizon

```
rly tx transfer kichain-t-3 groot-011 1000000utki rizon1jf8azthwxx8qmyqw8w0zh02p5pz2jmzw7uexxq --path transfer            
```
https://ki.thecodes.dev/tx/0FCBF06FEE26D9B1051D12E89F344A0E9F4118B77C894DAD58A69F5C1C731474
https://ki.thecodes.dev/tx/60BB5EC3CDEFCF9B3ACD550E0763B018BFE91D5769F8926F1D1578A8963498C2
  
We have performed IBC transaction between Rizon and Kichain blockchains.
