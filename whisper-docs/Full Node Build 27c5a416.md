---

Original guide can be found on the Desmos Website:

[Setup | Desmos documentation](https://docs.desmos.network/fullnode/setup "Setup | Desmos documentation")

Snapshot Link / Resources:

<https://nodejumper.io/desmos>

# Install pre-requisites

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
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
rm -rf go1.19.5.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

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

# Option 1: Build Daemon from source (Normal/LevelDB)

```shell
git clone https://github.com/desmos-labs/desmos.git
cd desmos
git checkout v2.4.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
desmos version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=desmos-mainnet
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME="desmos-0"
```

## Set minimum gas prices

In `$HOME/.desmos/config/app.toml`, set minimum gas prices:

```shell
nano ~/.desmos/config/app.toml

# update gas prices
minimum-gas-prices = "0.025udsm"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
desmos init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.desmos/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/desmos-labs/mainnet/main/genesis.json > ~/.desmos/config/genesis.json
```

This will replace the genesis file created using `desmos init` command with the mainnet `genesis.json`.

Confirm the Genesis file is correct:

```shell
jq -S -c -M '' ~/.desmos/config/genesis.json | shasum -a 256

# should output
619c9462ccd9045522300c5ce9e7f4662cac096eed02ef0535cca2a6826074c4  -
```

## Add Seed nodes

In this section, you will add your seed servers to ensure your validator node has the addresses of all new validators that come online.

Replace the `seeds` in `~/.desmos/config/config.toml` with:

```shell
seeds = "9bde6ab4e0e00f721cc3f5b4b35f3a0e8979fab5@seed-1.mainnet.desmos.network:26656,5c86915026093f9a2f81e5910107cf14676b48fc@seed-2.mainnet.desmos.network:26656,45105c7241068904bdf5a32c86ee45979794637f@seed-3.mainnet.desmos.network:26656"
```

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to set up cosmovisor and start the node.

---

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
desmos keys add WhisperNode
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

## Syncing the Node (State Sync + Snapshot)

**1. RPC Server**

```shell
http://rpc1.nodejumper.io:32657
```

**2. Peers**

```shell
f090ead239426219d605b392314bdd73d16a795f@rpc1.nodejumper.io:32656
```

```shell
peers="f090ead239426219d605b392314bdd73d16a795f@rpc1.nodejumper.io:32656"
sed -i -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.desmos/config/config.toml
```

**3. Snapshot** (height: 3,223,316; size: 3.1GB; pruning: custom/100/0/10, indexer: null)

```shell
#Install lz4, if needed
sudo apt update && sudo apt install snapd -y && sudo snap install lz4
```

```shell
# stop cosmovisor (if running) and reset data (if existing node)
sudo systemctl stop cosmovisor && desmos unsafe-reset-all

# remove existing data dir
rm -rf ~/.desmos/data && cd ~/.desmos

# setup snapshot
SNAP_NAME=$(curl -s https://snapshots1.nodejumper.io/desmos/ | egrep -o ">desmos-mainnet.*\.tar.lz4" | tr -d ">")
curl https://snapshots1.nodejumper.io/desmos/${SNAP_NAME} | lz4 -dc - | tar -xf -

# start cosmovisor
sudo systemctl restart cosmovisor
sudo journalctl -u desmosd -f --no-hostname -o cat
```

**4. State Sync**

```shell
# stop cosmovisor & reset data

sudo systemctl stop cosmovisor && desmos unsafe-reset-all

# set SNAP env vars

SNAP_RPC="https://desmos.nodejumper.io:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="f090ead239426219d605b392314bdd73d16a795f@desmos.nodejumper.io:32656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.desmos/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.desmos/config/config.toml

# restart / sync
sudo systemctl restart cosmovisor && sudo journalctl -u cosmovisor -f --no-hostname -o cat
```

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
desmos tx staking create-validator \
  --amount 1000000udsm \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(desmos tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025udsm \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
desmos tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.desmos/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
