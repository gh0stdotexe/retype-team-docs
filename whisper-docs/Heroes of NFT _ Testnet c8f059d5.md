Original Guide:

[Hero Testnet - Heroes of Nft](https://docs.heroesofnft.com/hero-testnet "Hero Testnet - Heroes of Nft")

Axelar Guide:

[Heroes of NFT | Axelar Network](https://docs.axelar.dev/validator/external-chains/hero#prerequisites "Heroes of NFT | Axelar Network")

Block Explorer (Testnet):

<br>

---

## Prerequisites

- [Setup your Axelar validator](https://docs.axelar.dev/validator/setup)
- Minimum hardware requirements: 8 AWS vCPU+, 16GB RAM, 500GB+ free storage space.
- MacOS or Ubuntu 18.04+
- Golang 1.18.1+
- [Official Documentation](https://docs.heroesofnft.com/hero-testnet)

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

Unless you want to configure them in a non-standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

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

---

## Install AvalancheGo

Heroes of NFT runs with `avalanche-go` version `v1.9.1` and `subnet-evm` `v0.4.2`. For any parameters, see either the official documentation mentioned in the prerequisites or the network information section below.

### 1. Build avalanche-go and subnet-evm

```shell
cd $GOPATH
mkdir -p src/github.com/ava-labs
cd src/github.com/ava-labs
git clone git@github.com:ava-labs/avalanchego.git
cd avalanchego
git checkout v1.9.1
./scripts/build.sh
​
# download and compile the subnet-evm binary
cd $GOPATH/src/github.com/ava-labs
git clone git@github.com:ava-labs/subnet-evm.git
cd subnet-evm
git checkout v0.4.2
./scripts/build.sh $GOPATH/src/github.com/ava-labs/avalanchego/build/plugins/nyfSdRoaSxyQUqMMQAVNaGR2bin6HRLC1yrRdEZRpfFrDiUk8
```

### 2. Launch Avalanche Node with Hero Subnet

```shell
./build/avalanchego --network-id=fuji --whitelisted-subnets 2q5qfMjarA3PJDEe594LFY2sHqNuD95CAkhkQTMY4Z6aHndgXT
```

Alternatively, instead of running `avalanchego` with command flags, the conf file `~/.avalanchego/configs/node.json` can also be edited:​

```shell
 {
   “http-host”: “127.0.0.1”,
   “http-port”: 9650,
   “network-id”: “fuji”,
   “whitelisted-subnets”: “2q5qfMjarA3PJDEe594LFY2sHqNuD95CAkhkQTMY4Z6aHndgXT”
 }
```

Now you should be synchronizing on the Avalanche Testnet network.

While the node is launching and catching up, search for logs pertaining to subnet ID `2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP`.​

```shell
[10-18|23:11:50.225] INFO <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> bootstrap/bootstrapper.go:115 starting bootstrapper
[10-18|23:11:50.225] INFO chains/manager.go:266 creating chain {"chainID": "ZyjdHoe9nbkVsUmuJEeWeFbtfAWKCsuZoTJ9WVUbbSLsc1aHX", "vmID": "nyfSdRoaSxyQUqMMQAVNaGR2bin6HRLC1yrRdEZRpfFrDiUk8"}
​
​
INFO [10-18|23:11:50.213] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/eth/backend.go:161: ---------------------------------------------------------------------------------------------------------------------------------------------------------
INFO [10-18|23:11:50.213] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/eth/backend.go:162:
INFO [10-18|23:11:50.214] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/eth/backend.go:195: Initialising Ethereum protocol network=17771 dbversion=8
INFO [10-18|23:11:50.217] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:506: Loaded most recent local header number=43737 hash=4085c2..936d3f age=10h1m10s
INFO [10-18|23:11:50.217] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:507: Loaded most recent local full block number=43737 hash=4085c2..936d3f age=10h1m10s
INFO [10-18|23:11:50.218] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:1438: Loaded Acceptor tip hash=4085c2..936d3f
INFO [10-18|23:11:50.218] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:1446: Skipping state reprocessing root=2de9eb..45c806
INFO [10-18|23:11:50.218] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:1420: Initializing snapshots async=true rebuild=true headHash=4085c2..936d3f headRoot=2de9eb..45c806
INFO [10-18|23:11:50.219] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/blockchain.go:373: Starting Acceptor queue length=64
INFO [10-18|23:11:50.220] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/core/tx_pool.go:506: Transaction pool price threshold updated price=0
[10-18|23:11:50.223] INFO <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> proposervm/vm.go:359 block height index was successfully verified
[10-18|23:11:50.223] INFO <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> snowman/transitive.go:80 initializing consensus engine
INFO [10-18|23:11:50.224] <2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP Chain> github.com/ava-labs/subnet-evm/plugin/evm/vm.go:746: Enabled APIs: eth, eth-filter, net, web3, internal-eth, internal-blockchain, internal-transaction
[10-18|23:11:50.225] INFO server/server.go:272 adding route {"url": "/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP", "endpoint": "/rpc"}
[10-18|23:11:50.225] INFO server/server.go:272 adding route {"url": "/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP", "endpoint": "/ws"}
```

## Test your Heroes of NFT RPC connection

Once your node is fully synced, you can run a cURL request to see the status of your node.

The EVM RPC should be running on your machine, at the URL:

```shell
http://127.0.0.1:9650/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc
```

Ensure that it is running by executing the following curl command.​

```shell
curl --silent http://127.0.0.1:9650/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc -X POST -H "Content-Type: application/json" --data '{"method":"eth_getBlockByNumber","params":["latest", false],"id":1,"jsonrpc":"2.0"}'

# If the node is successfully synced, the output from above will print

{"jsonrpc":"2.0","id":1,"result":{"baseFeePerGas":"0x59682f000","blockGasCost":"0x0","difficulty":"0x1","extraData":"0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","gasLimit":"0xf42400","gasUsed":"0x39765","hash":"0x4085c26854d2dcc13ca2c1f051bc1fd097b11ca3b1bd8049e30f0e9ccb936d3f","logsBloom":"0x00040000000000000040002020000000200000000002000000000008000040000000000000000000000000001000000000000000000000000000000000200000000000004000000000000008200000000000000000000000000000000020000000000000020000900000000000000800000000000000000000000010000400000000000000000100000100000000000001004000000000000400800000000000020000000000000000000000000000000020000080000000000000000000000000000002000000100000000000000000000000000000000000000000020020020010000000000000000000000000000000008000000000020400000008000000","miner":"0x0100000000000000000000000000000000000000","mixHash":"0x0000000000000000000000000000000000000000000000000000000000000000","nonce":"0x0000000000000000","number":"0xaad9","parentHash":"0xbf7122548ba346e612ec1d11b90028f90d26ee80afa5a48e9b4609bfd2f24c38","receiptsRoot":"0x3073c7035f0cafaf4f4884e00b94626675a55319da4d3a7ebbcab6989c4fb3f8","sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347","size":"0x332","stateRoot":"0x2de9ebfa56271c270b88f5fc90b91f6662c317b4b5f61cce5a689b6dc545c806","timestamp":"0x634e97c0","totalDifficulty":"0xaad9","transactions":["0xff632852e621a58bc249a59a6be87e2ca16d24727c18910807a7ab2735a39df1"],"transactionsRoot":"0x5cd072279c7afce34b659dd616f8a94439cccfe5ce8a513fef687fd1f8badce0","uncles":[]}}
```

To add an alias for the subnet, edit `~/.avalanchego/vms/aliases.json`:​

```shell
{
  "nyfSdRoaSxyQUqMMQAVNaGR2bin6HRLC1yrRdEZRpfFrDiUk8": ["hero", "herovm", "hvm"]
}
```

### EVM RPC endpoint URL

Axelar Network will be connecting to hero subnet, which is the EVM-compatible blockchain, so your `rpc_addr` should be exposed in this format:

```shell
http://IP:9650/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc
```

Example: [`http://192.168.192.168:9650/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc`](http://192.168.192.168:9650/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc)

---

## Network Information

### HERO Subnet IDs

```shell
VM ID: nyfSdRoaSxyQUqMMQAVNaGR2bin6HRLC1yrRdEZRpfFrDiUk8
Subnet ID: 2q5qfMjarA3PJDEe594LFY2sHqNuD95CAkhkQTMY4Z6aHndgXT
Chain ID: 2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP
```

### Subnet parameters:

```shell
Network ID: 17771
Chain ID: 17771
Block Gas Limit: 16,000,000
10s Gas Target: 24,000,000
Min Fee: 24 GWei
Target Block Rate: 2s (Same as C-Chain)
```

### Validators

```shell
Validator 1 ID (Authenticated): NodeID-J5nWRXnBpAXxFq6LvcA6LJ2JYc1hmqbDY
Validator 1 IP: 79.143.180.115
​
Validator 2 ID: NodeID-5c4Ckmrb7ZqxdnqAZxQV46zLsXRsty7oi
Validator 2 IP: 66.94.125.207
Domain: herotestnet.heroesofnft.com
```

### Metamask Params

```shell
Network Name: Hero Testnet
RPC URL: https://herotestnet.heroesofnft.com:443/ext/bc/2kdzD7eps9QRC449zypVGBbrzkwFrefLzmt24tS8MxmvEBuWvP/rpc
Chain ID: 17771
Symbol: HRM
Explorer: https://testnet.avascan.info/blockchain/hero/
```

### Genesis file

```shell
{
  "config": {
    "chainId": 17771,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x2086799aeebeae135c246c65021c82b4e15a2c451340993aacfd2751886514f0",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "muirGlacierBlock": 0,
    "subnetEVMTimestamp": 0,
    "feeConfig": {
      "gasLimit": 16000000,
      "targetBlockRate": 2,
      "minBaseFee": 24000000000,
      "targetGas": 24000000,
      "baseFeeChangeDenominator": 36,
      "minBlockGasCost": 0,
      "maxBlockGasCost": 10000000,
      "blockGasCostStep": 500000
    },
    "contractDeployerAllowListConfig": {
      "blockTimestamp": 0,
      "adminAddresses": ["0x4464bbe40cc1f1184bda0d43c010298fda3d8d7e"]
    },
    "contractNativeMinterConfig": {
      "blockTimestamp": 0,
      "adminAddresses": ["0x4464bbe40cc1f1184bda0d43c010298fda3d8d7e"]
    }
  },
  "nonce": "0x0",
  "timestamp": "0x0",
  "extraData": "0x",
  "gasLimit": "0xf42400",
  "difficulty": "0x0",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "4464bbe40cc1f1184bda0d43c010298fda3d8d7e": {
      "balance": "0x52b7d2dcc80cd2e4000000"
    }
  },
  "airdropHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "airdropAmount": null,
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "baseFeePerGas": null
}
```

## More Information on Avalanche Subnets

- <https://docs.avax.network/nodes/build/run-avalanche-node-manually>
- <https://docs.avax.network/subnets/create-a-fuji-subnet>
- <https://docs.avax.network/subnets/setup-dfk-node>
- <https://docs.avax.network/nodes/maintain/subnet-configs>

<br>
