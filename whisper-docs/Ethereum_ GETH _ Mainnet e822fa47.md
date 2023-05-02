Original Guide:

[Ethereum | Axelar Network](https://docs.axelar.dev/validator/external-chains/ethereum "Ethereum | Axelar Network")

Block Explorer (Mainnet):

<https://etherscan.io/>

---

## Prerequisites

- [Setup your Axelar validator](https://docs.axelar.dev/validator/setup)
- Minimum hardware requirements: CPU with 2+ cores, 4GB RAM, 600GB+ free storage space.
- macOS or Ubuntu 18.04+
- [Official Documentation](https://geth.ethereum.org/docs/getting-started)

## Install Geth

In this guide, we will be installing `Geth` with the built-in launchpad PPAs (Personal Package Archives) on Ubuntu. If you are on a different OS, please refer to the [official Documentation](https://geth.ethereum.org/docs/getting-started).

### 1. Enable launchpad repository

```shell
sudo apt install software-properties-common -y
sudo add-apt-repository -y ppa:ethereum/ethereum
```

### 2. Install the latest version of go-Ethereum:

```shell
sudo apt-get update -y
sudo apt-get install ethereum -y
```

### 3. Install a consensus layer :

To sync after the latest merge in the Ropsten network geth nodes should run a consensus client to be able to keep in sync with the chain. The list of consensus clients can be found at <https://ethereum.org/en/developers/docs/nodes-and-clients/#consensus-clients>

Users can opt for any Consensus client. To install Prysm as a Consensus client

```shell
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
export USE_PRYSM_VERSION=v3.1.2

#Download the Ropsten network genesis file
wget https://github.com/eth-clients/merge-testnets/raw/main/ropsten-beacon-chain/genesis.ssz

# generate JWT secret
./prysm.sh beacon-chain generate-auth-secret

# start prysm consensus client (using Tmux)
tmux new
./prysm.sh beacon-chain --http-web3provider=http://localhost:8551 --mainnet --jwt-secret=/home/whispernode/prysm/jwt.hex --block-batch-limit=64
```

## Run geth through systemd

### 1. Create systemd service file

After installation of `go-ethereum`, we are now ready to start the `geth` process but in order to ensure it is running in the background and auto-restarts in case of a server failure, we will set up a service file with systemd.

<br>

Note: In the service file below you need to replace `$USER` and path to `geth`, depending on your system configuration.

```shell
sudo nano /etc/systemd/system/geth.service

[Unit]
Description=Ethereum Node
After=network.target

[Service]
User=whispernode
Type=simple
ExecStart=/usr/bin/geth --syncmode "snap" --http --http.api=eth,net,web3,engine --http.vhosts * --http.addr 0.0.0.0 --authrpc.jwtsecret=/home/whispernode/prysm/jwt.hex
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

<br>

If you would like to run a node on the Testnet instead (Ropsten), you need to add the `--ropsten` flag to the configuration above.

### 2. Enable and start the geth service

```shell
sudo systemctl enable geth
sudo systemctl daemon-reload
sudo systemctl start geth
```

If everything was set-up correctly, your Ethereum node should now be starting the process of synchronization. This will take several hours, depending on your hardware.To check the status of the running service or to follow the logs, you can use:

```shell
sudo systemctl status geth
sudo journalctl -u geth -f
```

## Test your Ethereum RPC connection

Alternatively, you can now also use the Geth JavaScript console and check the status of your node by attaching it to your newly created `geth.ipc`. Don't forget to replace $USER and path, depending on your server configuration.

```shell
geth attach ipc:/root/.ethereum/geth.ipc
eth.syncing

#Testnet
#geth attach ipc:/root/.ethereum/ropsten/geth.ipc
```

Once your node is fully synced, the output from above will say `false`. To test your Ethereum node, you can send an RPC request using `cURL`

```shell
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
--data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

If you are testing it remotely, please replace `localhost` with the IP or URL of your server.

### EVM RPC endpoint URL

In order for Axelar Network to connect to your Ethereum node, your `rpc_addr` should be exposed in this format:

```shell
http://172.255.141.244:8545
```

---

## Upgrading GETH

You can view the latest GETH versions here:

[Downloads | Go Ethereum](https://geth.ethereum.org/downloads/ "Downloads | Go Ethereum")

Updating an existing Geth installation to the latest version can be achieved by stopping the node and running the following commands:

```shell
sudo systemctl stop geth
sudo apt-get update
sudo apt-get install ethereum
sudo apt-get upgrade geth
```

When the node is started again, Geth will automatically use all the data from the previous version and sync the blocks that were missed while the node was offline.
