Original Guide:

[Arbitrum (ETH) Blockchain Explorer](https://arbiscan.io/ "Arbitrum (ETH) Blockchain Explorer")

Arbitrum Explorer (Mainnet):

[Arbitrum | Axelar Network](https://docs.axelar.dev/validator/external-chains/arbitrum "Arbitrum | Axelar Network")

---

## Prerequisites

- [Setup your Axelar validator](https://docs.axelar.dev/validator/setup)
- [Setup your Ethereum node](https://docs.axelar.dev/validator/external-chains/ethereum/)
- Minimum hardware requirements: CPU with 2+ cores, 4GB RAM, 600GB+ free storage space.
- MacOS or Ubuntu 20.04+
- [Official Documentation](https://developer.offchainlabs.com/node-running/running-a-node)

## Steps

1. Install Docker
2. Install Arbitrum image
3. Configure vald

### Install docker

```shell
sudo apt update && sudo apt install curl jq -y < "/dev/null"
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
```

### Install Arbitrum node

```shell
mkdir -p $HOME/data/arbitrum
chmod -fR 777 $HOME/data/arbitrum
```

Now, you will see this flag in the command below `--l1.url <YOUR_ETH_RPC_URL>` this means that your arbitrum node needs a synced Ethereum node. Please provide the RPC URL of a synced Ethereum node with this flag.

⚠️

Please avoid using 3rd party providers like alchemy, infura, etc. These providers have a specific request limit, and your node can throw 100s of thousands of requests while trying to sync.

**MAINNET**

```shell
sudo docker run --rm -it -d -v /home/whispernode/data/arbitrum:/home/whispernode/.arbitrum -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/nitro-node:v2.0.7-10b845c --l1.url http://116.202.118.157:8545 --l2.chain-id=42161 --http.api=net,web3,eth,debug --http.corsdomain=\* --http.addr=0.0.0.0 --http.vhosts=\* --init.url="https://snapshot.arbitrum.io/mainnet/nitro.tar"
```

### Verify sync status

To verify if your node is in sync you can check the latest block from the explorer. Compare the block explorer (mainnet) against where your node is currently at using the following command:

```shell
sudo docker ps -q | xargs -L 1 sudo docker logs --tail 10 -f
```

- [Mainnet Explorer](https://arbiscan.io/)

### Test RPC connection

Once your node is fully synced, the output from above will say `false`. To test your Arbitrum RPC node, you can send an RPC request using `cURL`

```shell
curl -X POST http://localhost:8547 -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

If you are testing it remotely, please replace `localhost` with the IP or URL of your server.

### Configure vald

In order for `vald` to connect to your Arbitrum node, your `rpc_addr` should be exposed in vald's `config.toml`

**MAINNET**

```shell
[[axelar_bridge_evm]]
name = "arbitrum"
l1_chain_name = "Ethereum"
rpc_addr = "http://51.81.107.151:8547"
start-with-bridge = true
```

<br>
