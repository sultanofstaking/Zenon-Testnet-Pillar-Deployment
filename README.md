# Zenon-Testnet-Pillar-Deployment 4.0
## WARNING: I created this guide as an experiment for those setting up a pillar for the first time on 4.0 manually (without the deployer) and to help current Zenon testnet pillar holders migrate off their non-functioning pillars to a fresh install of 4.0. Full disclosure I have not migrated myself, I did a new install of 4.0 manually for SultanOfStaking pillar (i.e., registered 1st time w/ new producer address) https://explorer.znn.space/pillar/z1qpgdtn89u9365jr7ltdxu29fy52pnzwe4fl7zc . The steps below are my own and not endorsed by the Zenon team. This is being done in a very agile way, MVP of an experiment I expect our cross-functional community will participate and provide feedback so we can continuously improve. Please use the issue feature for fixes here. If you need to reach me 1:1 DM me on TG @SultanOfStaking - Try at your own risk. 

Pillar Node requirements
- Hardware: CPU >= 2 cores, RAM >= 2 GB, Storage >= 20 GB #free space; recommended Network 1Gbps dedicated bandwidth
- Software: Linux distros e.g. Ubuntu 20.04 LTS/Debian/Centos; #recommended NTP configuration, Watchdog service

## Pillar Install (fresh install on new VPS)
If you are using this guide because you have a 3.0 version that is not producing momentum or tried the update to 4.0 and failed. Before attempting to launch and migrate to a new pillar you need to be sure your old pillar is stoppped.

`sudo systemctl stop znnd.service`

Now spin up your new pillar with your provider of choice and follow the steps below.

Install prereqs

`sudo apt install unzip`

`sudo apt install wget`

`sudo apt install gnupg`

Download bundle

`cd ~ && wget https://testnet.znn.space/downloads/v4.0.0/znn-public-incentivized-testnet-bundle-linux-amd64.zip`

Check testnet Quickstart https://testnet.znn.space/#!quickstart.md on how to verify bundle

Extract package

`unzip znn-public-incentivized-testnet-bundle-linux-amd64.zip`

Ensure all was extracted ok

`ls -lah`

You should see znn-cli and znnd

Make files executable

`chmod +x znnd`

`chmod +x znn-cli`

Enable RPC

`./znn-cli enableRPC`

#If you are setting up a pillar for the first time (not migrating) follow these steps
Generate producer address on vps (replace yourComplexPassphrase with a new password for keystore)

`./znn-cli wallet.createNew yourComplexPassphrase`

Register via syrius with vps address above as producer address

#If you are migrating your old pillar to a new VPS follow these steps
Get mnemonics for the producer and syrius addresses on your old pillar

`./znn-cli wallet.dumpMnemonic --keyStore yourproducerkeystore --passphrase yourComplexPassphrase`

`./znn-cli wallet.dumpMnemonic --keyStore yoursyriuskeystore --passphrase yourComplexPassphrase`

Import producer address from your old pillar to new vps via mnemonic (you need the mnemonic from the producer address you registered the pillar from and "yourComplexPassphrase" should be the passphrase you used when first setting up the producer address (be sure to put spaces between each of the words in your mnemonic)

`./znn-cli wallet.createFromMnemonic “your mnemonic” yourComplexPassphrase`

#Remaining steps will be the same for first time pillars or migrating pillars
Import syrius address to new vps via mnemonic (you need the  mnemonic from they syrius address you registered the pillar from, "yourComplexPassphrase" should be your syrius password, be sure to put spaces between each of the words in your mnemonic)

`./znn-cli wallet.createFromMnemonic “your mnemonic” yourComplexPassphrase`

Update config

`nano ~/.znn/config.json`

Paste the following in your config. Be sure to replace KeyStore & Password with your info

```
{
    "RPC": {
        "EnableHTTP": true,
        "EnableWS": true,
        "EnableIPC": true,
        "Endpoints": [
            "ledger",
            "embedded",
            "pow.links",
            "stats",
            "subscribe"
        ]
    },
    "Pillar": {
        "KeyStorePath": "producer address",
        "KeyStorePassword": "yourpassword",
        "ProducerAddress": "0:producer address"
    }
}
```

Save and exit using ctrl+o, enter, ctrl+x

Set up znnd.service

`sudo nano /etc/systemd/system/znnd.service`

Paste the following in your znnd.service. If your znnd is in another path specify that in the ExecStart line
```
[Unit]
Description=znnd

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/znnd
ExecStop=/usr/bin/pkill -9 znnd
Restart=on-failure
PrivateTmp=true
TimeoutStopSec=15s
TimeoutStartSec=15s
StartLimitInterval=0s
StartLimitBurst=5

[Install]
WantedBy=multi-user.target
```
Save and exit using ctrl+o, enter, ctrl+x

Open another terminal windows logged into your node and run

`sudo journalctl -f -u znnd.service`

In your primary reload the znnd.service daemon

`sudo systemctl daemon-reload`

Then start znnd.service

`sudo systemctl start znnd.service`

Your new terminal window should display logs during the startup. You should see it start without errors, get “znnd successfully started” and “Pillar detected!” messages, and see your node begin to sync and ultimately produce momentums. If this is not the case look for errors in the startup process then refer to my Node Tips repository to troubleshoot or ask the community in TG. The most common errors have to do with the data paths.

## If you found helpful no need to donate, but delegation to SultanOfStaking pillar would be appreciated https://explorer.znn.space/pillar/z1qpgdtn89u9365jr7ltdxu29fy52pnzwe4fl7zc

## If you are getting errors in your logs go here https://github.com/sultanofstaking/Zenon-Testnet-Node-Tips
