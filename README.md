# ibc-transactions-for-kichain

!(https://ibb.co/KzkfB1S)

In this guide we will go though the process of setting up a Relayer and sending tx's with IBC. 
Repeater node is set up on Rizon node. You can also set it on the Kichain node, or remote node but you need access to both nodes.

The chains we are using are groot-011 for Rizon and kichain-t-3 for Kichain.

First command is to make sure IBC transactions are enabled.
```
rizond q ibc-transfer params
```
The output should provide:
```
receive_enabled: true 
send_enabled: true  
```
We are using Relayer v1.0.0 from official source
```
git clone https://github.com/cosmos/relayer.git
cd relayer
make install
cd
rly version
```
Correct output is 
```
version: 1.0.0-rc1–152-g112205b
```
Now we will need to initialize a repeater with this command
```
rly config init
```
rly_config directory will store config files
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
Add settings to config file
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
Change standart timeout to 30 seconds
```
nano ~/.relayer/config/config.yaml
timeout: 30s  
```
Fund the wallets in each network. Check that you have coins in each wallet.
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

Now by calling this command you will be able to see that we are ready to send transactions
```
rly paths list -d
```
```
0: transfer             -> chns(✔) clnts(✔) conn(✔) chan(✔) (groot-011:transfer<>kichain-t-4:transfer)
```

You can perform transactions with the command
```
rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags] 
```
More "friendly" command will be like
```
rly tx transfer groot-011 kichain-t-3 1000000uatolo tki1jf8azthwxx8qmyqw8w0zh02p5pz2jmzwnu73r6 --path transfer
```
We are transfering tokens from Rizon to Kichain
  
Now we can see our transaction in explorer
https://dev.mintscan.io/rizon/txs/634104F5AC84FDD8446903C93F7E8E8A48429433697364968FD4C06476DA98EA
https://dev.mintscan.io/rizon/txs/341DF3CA9DBE12E4BD557B8A29581F89FB4B4DAD2C0FC60160577DAB12F9D709

Now send tokens in the opposite direction

```
rly tx transfer kichain-t-3 groot-011 1000000utki rizon17709aq0l4mzw5vh54f7rj4zw6xy45yfjfc9rmp --path transfer            
```
https://ki.thecodes.dev/tx/0FCBF06FEE26D9B1051D12E89F344A0E9F4118B77C894DAD58A69F5C1C731474
https://ki.thecodes.dev/tx/60BB5EC3CDEFCF9B3ACD550E0763B018BFE91D5769F8926F1D1578A8963498C2
  
We have performed IBC transaction between Rizon and Kichain blockchains.
