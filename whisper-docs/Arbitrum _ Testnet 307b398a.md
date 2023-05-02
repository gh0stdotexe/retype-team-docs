Original Guide:

[Running a Node | Arbitrum Documentation Center](https://developer.arbitrum.io/node-running/running-a-node "Running a Node | Arbitrum Documentation Center")

Testnet Explorer:

[GOERLI Arbitrum (ETH) Blockchain Explorer](https://goerli.arbiscan.io/ "GOERLI Arbitrum (ETH) Blockchain Explorer")

## Prerequisites

- [Setup your Axelar validator](https://docs.axelar.dev/validator/setup)
- [Setup your Ethereum node](https://docs.axelar.dev/validator/external-chains/ethereum/)
- Minimum hardware requirements: CPU with 2+ cores, 4GB RAM, 600GB+ free storage space.
- MacOS or Ubuntu 20.04+
- [Official Documentation](https://developer.offchainlabs.com/node-running/running-a-node)

---

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
mkdir -p /home/whispernode/data/arbitrum
chmod -fR 777 /home/whispernode/data/arbitrum
```

Now, you will see this flag in the command below `--l1.url <YOUR_ETH_RPC_URL>` this means that your arbitrum node needs a synced Ethereum node. Please provide the RPC URL of a synced Ethereum node with this flag.

⚠️

Please avoid using 3rd party providers like alchemy, infura, etc. These providers have a specific request limit, and your node can throw 100s of thousands of requests while trying to sync.

**GOERLI TESTNET**

```shell
sudo docker run --rm -it -d -v /home/whispernode/data/arbitrum:/home/whispernode/.arbitrum -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/nitro-node:v2.0.7-10b845c --l1.url https://goerli-rpc.polkachu.com/ --l2.chain-id=421613 --http.api=net,web3,eth,debug --http.corsdomain=\* --http.addr=0.0.0.0 --http.vhosts=\*
```

### Verify sync status

To verify if your node is in sync you can check the latest block from the explorer. Compare it with what you have using:

```shell
sudo docker ps -q | xargs -L 1 sudo docker logs --tail 10 -f
```

- [Testnet Explorer](https://goerli.arbiscan.io/)

### Test RPC connection

Once your node is fully synced, the output from above will say `false`. To test your Arbitrum RPC node, you can send an RPC request using `cURL`

```shell
curl -X POST http://localhost:8547 -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

If you are testing it remotely, please replace `localhost` with the IP or URL of your server.

### Configure vald

In order for `vald` to connect to your Arbitrum node, your `rpc_addr` should be exposed in vald's `config.toml`

**GOERLI TESTNET**

```shell
[[axelar_bridge_evm]]
name = "arbitrum"
l1_chain_name = "ethereum-2"
rpc_addr = "http://51.81.245.71:8547"
start-with-bridge = true
```

<br>
