Original Guide:

[Kava | Axelar Network](https://docs.axelar.dev/validator/external-chains/kava "Kava | Axelar Network")

Block Explorer (Testnet):

[Interchain Explorer by Cosmostation](https://testnet.mintscan.io/kava-testnet "Interchain Explorer by Cosmostation")

---

This guide presents you with the Kava Node setup for both Testnet and Mainnet, as well as the integration with Axelar Network.

## Prerequisites

- [Setup your Axelar validator](https://docs.axelar.dev/validator/setup)
- Minimum hardware requirements: 4 core CPU, 8GB RAM
- MacOS or Ubuntu 18.04+
- Build-essential packages
- Golang 1.17+
- [Official Documentation](https://docs.kava.io/docs/participate/validator-node/)

# Install Go

Follow the instructions [here](https://golang.org/doc/install) to install Go.

For an Ubuntu LTS, we can use:

```shell
# find location of existing GO (if any)
which go
go version

# remove old GO if existing
sudo rm -rf /usr/local/go

# install updated GO
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm -rf go1.19.3.linux-amd64.tar.gz
```

Unless you want to configure them in a non-standard way, then set these in the `.zshrc` in the user's home (i.e., `~/`) folder.

```shell
nano ~/.zshrc
```

Add the “export Pathing” rules  at the bottom, and then save the file:

```shell
# add export PATHS below
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin:$GOBIN
```

After updating your `~/.zshrc` you will need to source it:

```shell
source ~/.zshrc
```

## Install Kava

```shell
git clone https://github.com/Kava-Labs/kava
cd kava
```

**TESTNET**

```shell
git checkout v0.19.5-testnet
make install

# verify version
kava version --long
```

## Initialize Kava Node and create necessary toml files

Use this hex generator to randomly generate a 32-bit node moniker (for security purposes):

[Best Random Number Generator by NumberGenerator.org](https://numbergenerator.org/hex-code-generator#!numbers=1\&length=32\&addfilters= "Best Random Number Generator by NumberGenerator.org")

```shell
kava init 501763AE568602C5E3948461E364D45E --chain-id kava_2221-16000
```

```shell
kava tendermint unsafe-reset-all --home $HOME/.kava
cd $HOME/.kava/config
rm genesis.json
```

Download Genesis File

```shell
wget https://raw.githubusercontent.com/Kava-Labs/kava-testnets/master/16000/genesis.json
```

Edit your `config.toml` file and add the following persistent peers.

```shell
persistent_peers = "0ecf6974e645d267e1c40223fabc59521229de5a@203.238.191.195:26656,19a109e7b1bdbe020db34d883fc740006fd96d9e@54.205.248.136:26656,a610e40ec9c15ceab183bdc71e94fbf9c4c91d70@54.91.207.93:26656,0264f7762530ef4e637f0fcb866bea0b2ea68dd9@34.198.224.121:26656,c22a30eee9f4873a7891603a567d5825db4583a4@65.21.232.149:26756,b2dc431ea24139d5d2d728d6edb546b3dd8ca8ec@54.85.1.24:26656,cb44d1deb4b1732adc1fb81e2bdeed097c5052c2@195.201.108.152:26656,2a0fd732f484647a34076d03501b5ade76a0d056@54.162.132.18:26656,9b2b83d1bccab3c388c379f6b7a19146ee9b4d8b@161.97.155.94:30656,5af375a8f1fb5879087ea573e30976f084f23769@167.235.244.61:26656,8be1950812fe31c91a63eab482d13276fb7cc4c7@95.217.214.138:26656,e2e9d76fb4658f1757d4b51ef34ebbd1e857d1c5@88.99.25.84:26656,b9f759ca8cf1179ac10b36e6f458c69bf5ee2b85@89.149.218.18:26656,0e7c323c8d558f3aeecee09a679f39f7842287ca@52.207.243.95:26656,ddcc5e1b405fbf5763c7c1b3932c44f3f0917a58@50.19.243.132:26656,220aa32bc7279a43a2ce9af0393945c357e88b27@86.111.48.129:26656,25b3e8aa0c0a7bb47710cb5be1a700c243da211c@3.93.65.190:26656,2972850c607f65354af82c75382fa1745d46b63a@65.21.126.182:26656,9d0bad73fbf1bc53b3e8b4b3c7873980b1843524@18.207.210.224:26656,547f3b5f5907339794f6231d71ac0bd2246bdcb9@65.108.124.57:21656,353ae6090dc1fffb66d55faef6b6133693907af4@3.83.193.144:26656,80f5c216dbb472b483a578154167953c3a2913bb@34.224.245.49:26656,0f7c1725cacc5be8000209bbce4be55b7b5d7fe5@18.232.239.76:26656,4b127af8f935836798fb24b451849c6a28b0bf26@222.106.187.14:54506,a3176f14632e2ce70b879fd7c74b3e63626559d2@54.156.38.39:26656,47b3b6a795febcf3de0b50a65d383a8573e26289@3.80.131.4:26656,8d25035ba3da76aa8257cea0a110ea586511312e@65.109.26.45:26656,3b21dbf61555cbbc3828fca3af26737b81104a32@86.111.48.130:26656,ed0212d3afc1d6c9a02b785ca1577e1ee6609672@52.205.115.127:26656,d4b23cf74b22ea4c8be862151486dd7301e0b84b@44.199.250.214:26656,0f3bcd2ac6f13522600a7a20861c4c5354bf518e@34.226.104.212:26656,b859111f93f4d461a6de749e865c7fef2ac912d8@139.162.210.87:26656"
```

Add minimum gas prices into `app.toml`

```shell
minimum-gas-prices = "0.025ukava;1000000000akava"
```

Do the rest of the port configuration as desired; The EVM configuration and ports can be found in the `app.toml` under the section “EVM Configuration”

---

## State sync script

**TESTNET RPC**

```shell
https://kava-testnet.chainode.tech:443
```

```shell
# create state sync script
nano statesync.sh
```

Add the content below and change the RPC var for the desired network

```shell
#!/bin/bash

RPC="https://kava-testnet.chainode.tech:443"

LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.kava/config/config.toml
```

Assign the correct access rights

```shell
chmod 700 statesync.sh
```

Execute the script

```shell
./statesync.sh
```

## Start Kava Node with systemd

```shell
sudo nano /etc/systemd/system/kvd.service

[Unit]
Description=KAVA daemon
After=network-online.target

[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/kava start
Restart=always
RestartSec=3
LimitNOFILE=16384

[Install]
WantedBy=multi-user.target

# enable + start service
sudo systemctl enable kvd
sudo systemctl daemon-reload
sudo systemctl start kvd
```

## Kava EVM Endpoint integration with Axelar

Once the Kava Node is fully synced, you can use the EVM RPC endpoint for the integration with Axelar Network.

```shell
[[axelar_bridge_evm]]
name = "Kava"
rpc_addr = "http://51.81.66.123:26657"
start-with-bridge = true
```

<br>
