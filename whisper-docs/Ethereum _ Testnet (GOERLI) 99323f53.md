Original Guide:

[axelar-docs/ethereum.md at main · axelarnetwork/axelar-docs](https://github.com/axelarnetwork/axelar-docs/blob/main/pages/validator/external-chains/ethereum.md "axelar-docs/ethereum.md at main · axelarnetwork/axelar-docs")

Block Explorer (Testnet):

<https://goerli.etherscan.io/>

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
sudo apt-get install software-properties-common -y
sudo apt-get update
sudo add-apt-repository ppa:ethereum/ethereum
```

### 2. Install the latest version of go-ethereum:

```shell
sudo apt-get update
sudo apt-get install ethereum -y
```

### 3. Install a consensus layer:

To sync after the latest merge in the `Ropsten` network `geth` nodes should run a consensus client to be able to keep in sync with the chain. The list of consensus clients can be found at <https://ethereum.org/en/developers/docs/nodes-and-clients/#consensus-clients>

Users can opt for any Consensus client. To install Prysm as a Consensus client

```shell
mkdir prysm && cd prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
export USE_PRYSM_VERSION=v3.1.1
#Download the Goerli network genesis file
https://github.com/eth-clients/eth2-networks/raw/master/shared/prater/genesis.ssz
#Generate a secret key for authentication
nano jwtsecret
# insert hex key into doc and save
eb1df008b7ea05be008b6a90e94834530f00659651bd536f46a1be415020b35b
#Start Local Prysm beacon chain 
./prysm.sh beacon-chain --http-web3provider=http://localhost:8551 --prater --jwt-secret=/home/whispernode/prysm/jwt.hex --genesis-state=./genesis.ssz
```

Refer:

[Nodes and clients | ethereum.org](https://ethereum.org/en/developers/docs/nodes-and-clients/#consensus-clients "Nodes and clients | ethereum.org")

[Configure for The Merge | Prysm](https://docs.prylabs.network/docs/prepare-for-merge "Configure for The Merge | Prysm")

\[<https://seanwasere.com/generate-random-hex/>]

## Run geth through systemd

### 1. Create a systemd service file

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
ExecStart=/usr/bin/geth --syncmode "snap" --http --http.api=eth,net,web3,engine --http.vhosts * --http.addr 0.0.0.0 --authrpc.jwtsecret=/home/whispernode/prysm/jwtsecret --goerli --override.terminaltotaldifficulty 50000000000000000
Restart=always
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

To test your Ethereum node, you can send an RPC request using `curl `to see if the chain is fully synced:

```shell
curl -X POST http://localhost:8545 -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

This should return `false` if the node is fully synced:

```shell
{"jsonrpc":"2.0","id":1,"result":false}
```

### EVM RPC endpoint URL

In order for Axelar Network to connect to your Ethereum node, your `rpc_addr` should be exposed in this format:

```shell
http://65.108.238.47:8545
```

Register Goerli as a Chain Maintainer

Run the following command to register `goerli` as the nexus chain-maintainer for the axelar testnet:

```shell
axelard tx nexus register-chain-maintainer ethereum-2 --from broadcaster --gas auto --gas-adjustment 1.4
```

[Overview | Axelar Network](https://docs.axelar.dev/validator/external-chains/overview "Overview | Axelar Network")
