# Initia

## Recommended Hardware Requirements

|   SPEC      |       Recommend          |
| :---------: | :-----------------------:|
|   **CPU**   |        4 Cores           |
|   **RAM**   |        16 GB             |
| **Storage** |        TB SSD            |
| **NETWORK** |        100 Mbps          |
|   **OS**    |        Ubuntu 22.04      |
|   **Port**  |        26656             | 

### Server preparing
```
cd $HOME && sudo apt install -y build-essential
```

### Install Go
```
ver="1.21.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile
```

### Download the binary file
```
git clone https://github.com/initia-labs/initia
cd initia
git checkout v0.2.10
make install
```

### Initialize node (change “tinhdv” when configuring)
```
 initiad init "your_moniker" --chain-id initiation-1
```

### Download genesis
```
 curl -Ls https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json > $HOME/.initia/config/genesis.json
```

### Setting up minimum-gas-prices, peers, seeds
```
 sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml
```
```
 PEERS="40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,841c6a4b2a3d5d59bb116cc549565c8a16b7fae1@23.88.49.233:26656,e6a35b95ec73e511ef352085cb300e257536e075@37.252.186.213:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,065f64fab28cb0d06a7841887d5b469ec58a0116@84.247.137.200:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,093e1b89a498b6a8760ad2188fbda30a05e4f300@35.240.207.217:26656,12526b1e95e7ef07a3eb874465662885a586e095@95.216.78.111:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i \
    -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" \
    -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
    "$HOME/.initia/config/config.toml"
```
```
 sed -i \
    -e "s/^pruning *=.*/pruning = \"custom\"/" \
    -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" \
    -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" \
    "$HOME/.initia/config/app.toml"
```

### Create a service file
```
 sudo tee /etc/systemd/system/initiad.service > /dev/null << EOF
[Unit]
Description=Initia node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Start the node
```
 sudo systemctl daemon-reload && \
 sudo systemctl enable initiad && \
 sudo systemctl restart initiad && \
 sudo journalctl -u initiad -f -o cat
```

### Create wallet  (change “wallet” when configuring)
1. Create a new key
```
 initiad keys add wallet
```
2. Restore an existing key using a mnemonic (optional)
```
 initiad keys add wallet --recover
```

### Update
1. Stop the node
```
 sudo systemctl restart initiad
```
2. Upgrade to the latest version
```
 cd initia
 git fetch --all
 git checkout v0.2.10
 make install
```
3. Start the node
```
 sudo systemctl enable initiad
 sudo systemctl start initiad
```
4. Check logs
```
 sudo journalctl -u initiad -f -o cat
```

### Useful commands
1. Check info node
```
 initiad status 2>&1 | jq .NodeInfo
```
2. Check synchronization
```
 initiad status 2>&1 | jq .SyncInfo
```
3. Check logs
```
 journalctl -u initiad -f -o cat
```
4. Check balance
```
 initiad q bank balances init...
```

### Backup Validator
```
 mkdir -p $HOME/backup/.initia
 cp $HOME/.initia/config/priv_validator_key.json $HOME/backup/.initia
```

### Register as Validator
```
 initiad tx mstaking create-validator \\
    --amount="<bond_amount>" \\
    --pubkey=$(initiad tendermint show-validator) \\
    --moniker="<your_moniker>" \\
    --identity="<keybase_identity>" \\
    --chain-id="<chain_id>" \\
    --from="<key_name>" \\
    --commission-rate="0.10" \\
    --commission-max-rate="0.20" \\
    --commission-max-change-rate="0.01"
```

### Remove Node
```
 cd $HOME
 sudo systemctl stop initia
 sudo systemctl disable initia
 sudo rm /etc/systemd/system/initia.service
 sudo systemctl daemon-reload
 sudo rm -f $(which initiad)
 sudo rm -rf $HOME/.initia
```
