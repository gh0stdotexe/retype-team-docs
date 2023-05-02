Original Guide:

[You need server with following requirements | Axelar Network](https://docs.axelar.dev/validator/external-chains/binance "You need server with following requierements | Axelar Network")

Block Explorer:

<https://www.bscscan.com/>

GitHub Repo:

<https://github.com/binance-chain/bsc/releases/download/v1.1.4/geth_linux>

---

# You need a server with the following requirements

2CPU/8GBRAM/200++GB SSD

## Upgrade your server

```shell
sudo apt update && sudo apt upgrade -y
```

```shell
sudo apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen unzip -y
```

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

Unless you want to configure them in a nonstandard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.zshrc
```

Add the "export Pathing" rules  at the bottom, and then save the file:

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

# Install BSC Node

```shell
git clone https://github.com/bnb-chain/bsc
cd bsc
make geth
```

## Download the config files

You can download the pre-built binaries from [release page](https://github.com/bnb-chain/bsc/releases/latest) or follow the instructions bellow

```shell
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.17/geth_linux
```

### MAINNET:

```shell
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.17/mainnet.zip
unzip mainnet.zip
```

```shell
sudo mv geth_linux /usr/bin/geth
sudo chmod +x /usr/bin/geth
rm -rf mainnet.zip
```

## Configure config.toml file

```shell
nano config.toml
```

Inside the `config.toml` found line `HTTPHost = "127.0.0.1"` and change IP from `127.0.0.1` on your IP address.

```shell
[Node]
IPCPath = "geth.ipc"
HTTPHost = "65.108.233.51"
NoUSB = true
InsecureUnlockAllowed = false
HTTPPort = 8545
HTTPVirtualHosts = ["localhost"]
HTTPModules = ["eth", "net", "web3", "txpool", "parlia"]
WSPort = 8546
WSModules = ["net", "web3", "eth"]
```

Scroll down and delete the following lines:

```shell
[Node.LogConfig]
FileRoot = ""
FilePath = "bsc.log"
MaxBytesSize = 10485760
Level = "info"
```

Save it by command `CTRL+X,Y`

## Write state genesis locally

```shell
geth --datadir node init genesis.json
```

## Configure a systemd service file

```shell
sudo nano /etc/systemd/system/bscd.service

[Unit]
Description=BSC
After=network-online.target

[Service]
User=whispernode
ExecStart=/usr/bin/geth --config home/whispernode/bsc/config.toml --datadir home/whispernode/bsc/node --ws --ws.origins '*'
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
```

```shell
sudo systemctl daemon-reload
sudo systemctl enable bscd
sudo systemctl start bscd
```

Check that the node starts syncing

```shell
journalctl -u bscd -f -n 100
```

## Sync status

Once your BSC node is fully synced, you can run a cURL request to see the status of your node: Please change `YOUR_IP_ADDRESS` on your IP.

```shell
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "eth_syncing", "params":[]}' http://65.108.233.51:8545
```

If the node is successfully synced, the output from above will print `{"jsonrpc":"2.0","id":1,"result":false}`

## Connect your BSC to Axelar

Axelar Network will be connecting to the EVM compatible `Binance`, so your `rpc_addr` should be exposed in this format:

### MAINNET:

Example:&#x20;

```shell
http://172.255.141.252:8545
```

---

## Updating BSC

If a new update has been released, then you can update with the following commands.

```shell
systemctl stop bscd
mkdir temp
cd temp
```

Check the latest [release page](https://github.com/bnb-chain/bsc/releases/latest). In this example the latest release is `v1.1.17`

```shell
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.17/geth_linux
```

```shell
mv geth_linux /usr/bin/geth
chmod +x /usr/bin/geth
```

```shell
cd ~
systemctl start bscd
```

Check that your BSC node works correctly

```shell
journalctl -u bscd -f -n 100
```

<br>
