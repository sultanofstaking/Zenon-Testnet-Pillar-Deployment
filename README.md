# Zenon-Testnet-Pillar-Deployment
# I created this guide to help Zenon testnet users set up a pillar. The steps below are my own and not endorsed by the Zenon team. Try at your own risk. If you found helpful no need to donate, but delegation to SultanOfStaking pillar would be appreciated https://explorer.znn.space/pillar/z1qpgdtn89u9365jr7ltdxu29fy52pnzwe4fl7zc

#Pillar Node requirements
#Hardware: CPU >= 2 cores, RAM >= 2 GB, Storage >= 20 GB #free space; recommended Network 1Gbps dedicated bandwidth
#Software: Linux distros e.g. Ubuntu 20.04 LTS/Debian/Centos; #recommended NTP configuration, Watchdog service

# Pillar Instal
#Instal prereqs

sudo apt install unzip

sudo apt install wget

sudo apt install gnupg

#Download bundle

cd ~ && wget https://testnet.znn.space/downloads/v4.0.0/znn-public-incentivized-testnet-bundle-linux-amd64.zip

#Check testnet Quickstart https://testnet.znn.space/#!quickstart.md on how to verify bundle

#Extract package

unzip znn-public-incentivized-testnet-bundle-linux-amd64.zip

#Ensure all was extracted ok

ls -lah

#You should see znn-cli and znnd

#Make files executable

chmod +x znnd 

chmod +x znn-cli

#Enable RPC

./znn-cli enableRPC

#Generate producer address address on vps

./znn-cli wallet.createNew yourComplexPassphrase

#Register via syrius with vps address above as producer address

#Import syrius address to vps via mnemonic

./znn-cli wallet.createFromMnemonic “your” mnemonic” yourComplexPassphrase

#Update config

nano ~/.znn/config.json

#Paste the following in your config. Be sure to replace KeyStore & Password with your info

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

#Save and exit using ctrl+o, enter, ctrl+x

#Set up znnd.service

sudo nano /etc/systemd/system/znnd.service

#Paste the following in your znnd.service. If your znnd is in another path specify that in the ExecStart line

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

#Save and exit using ctrl+o, enter, ctrl+x

#Open another terminal windows logged into your node and run

sudo journalctl -f -u znnd.service

#In your primary reload the znnd.service daemon

sudo systemctl daemon-reload

#Then start znnd.service

sudo systemctl start znnd.service

#Your new terminal window should display logs during the startup. You should see it start without errors, get “znnd successfully started” and “Pillar detected!” messages, and see your node begin to sync and ultimately produce momentums. If this is not the case look for errors in the startup process then refer to my Node Tips repository to troubleshoot or ask the community in TG. 
