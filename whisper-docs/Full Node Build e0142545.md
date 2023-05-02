---

Website:

[WhiteWhale Protocol](https://whitewhale.money/ "WhiteWhale Protocol")

GitHub Repo:

[GitHub - White-Whale-Defi-Platform/migaloo-chain: Migaloo...](https://github.com/White-Whale-Defi-Platform/migaloo-chain "GitHub - White-Whale-Defi-Platform/migaloo-chain: Migaloo is a cosmwasm-powered, permissionless blockchain for building decentralized applications.")

Snapshot:

[Whitewhale Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/whitewhale "Whitewhale Node Snapshot | Polkachu")

Block Explorer:

[Silk Nodes Dashboard](https://explorer.silknodes.io/whitewhale/staking "Silk Nodes Dashboard")

<br>

# Install prerequisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y
sudo apt-get install make build-essential gcc git jq chrony -y
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
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm -rf go1.20.linux-amd64.tar.gz
```

Unless you want to configure in a non-standard way, then set these in the `.zshrc` in the user's home (i.e., `~/`) folder.

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

After updating your `~/.profile` you will need to source it:

```shell
source ~/.zshrc
```

# Build Daemon from source

```shell
git clone https://github.com/White-Whale-Defi-Platform/migaloo-chain WhiteWhale
cd WhiteWhale
git checkout v1.0.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
migalood version --long
```

# Configuration of Shell Variables

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

```shell
migalood init 3FA0113B769EBB374B427CE8951B1530 --chain-id migaloo-1
```

### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```shell
curl https://raw.githubusercontent.com/White-Whale-Defi-Platform/migaloo-chain/release/v1.0.x/networks/mainnet/genesis.json > ~/.migalood/config/genesis.json 
```

### Configure Seed

Using a seed node to bootstrap is the best practice in our view. Alternatively, you can use [**addrbook**](https://polkachu.com/addrbooks/cerberus) or [**persistent\_peers**](https://polkachu.com/live_peers/cerberus).

```shell
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:13856"/' ~/.migalood/config/app.toml
```

## Setting Persistent Peers

In this section, you will add peers to begin communicating with the `WhiteWhale` blockchain.

```shell
persistent_peers="3ef97d0e832e9e1312da0e5217a9297dd7f4b900@135.181.215.62:26656,538b5c109a7b7d64ddb50b7d3de518321bc833c4@192.99.44.79:20756,ad9d79aba19b176117aa0c73e519ee66d205b6ea@135.181.223.115:26656"
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
migalood keys add WhisperNode --keyring-backend os
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

# Syncing the node

After starting the `migalood` daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block, and you are safe to proceed to upgrade to a validator node.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
migalood tx staking create-validator \
  --amount 1000000uwhale \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(migalood tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uwhale \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io ID, etc. To see a full list:

```shell
migalood tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.migalood/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
