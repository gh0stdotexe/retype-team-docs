Original Guide:

[testnets/uni-5 at main · CosmosContracts/testnets](https://github.com/CosmosContracts/testnets/tree/main/uni-5 "testnets/uni-5 at main · CosmosContracts/testnets")

Testnet Support by Polkachu:

<br>

---

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
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm -rf go1.20.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

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

After updating your `~/.profile` you will need to source it:

```shell
source ~/.zshrc
```

# Build Daemon from source

```shell
git clone https://github.com/CosmosContracts/juno
cd juno
git checkout v12.0.0-beta.1
make build && make install
```

To confirm that the installation has succeeded, you can run:

```shell
junod version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet, you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=uni-6
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME=<moniker-name>

# Example
MONIKER_NAME="WhisperNode-0"
```

## Set minimum gas prices

In `$HOME/.juno/config/app.toml`, set minimum gas prices:

```shell
nano ~/.juno/config/app.toml
minimum-gas-prices = "0.0025ujunox"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
junod init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.juno/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O genesis.json https://snapshots.polkachu.com/testnet-genesis/juno/genesis.json --inet4-only
mv genesis.json ~/.juno/config
```

This will replace the genesis file created using `junod init` command with the mainnet `genesis.json`.

## Set seeds

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.juno/config/config.toml`:

```shell
# seeds list
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:12656"/' ~/.juno/config/config.toml
```

---

## [Polkachu State-Sync Peer for JUNO UNI-3 Testnet](https://polkachu.com/testnets/juno/peers "Juno Testnet Node Peers | Polkachu")

When you state-sync, you might also consider adding Polkachu's state-sync peer to your **`persistent_peers`** setting in **`config.toml`**.

```shell
26709b3d89548a865ccfd2efac34ef3e9a5b2bc4@135.181.59.162:26656
```

<br>

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
junod keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
junod keys add <key-name> --recover

# Query the keystore for your public address
junod keys show <key-name> -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 92cfab7e.md>)

---

## [Polkachu State-Sync Server Setup](https://polkachu.com/testnets/juno/state_sync "Juno Testnet Node State-Sync | Polkachu")

Our state-sync RPC server for `JUNO` is:

```shell
https://juno-testnet-rpc.polkachu.com:443
```

## Instruction

We assume that you use Cosmovisor to manage your node. If you do not use Cosmovisor, you will need to customize the following instruction slightly.

Create a reusable shell script such as **`state_sync.sh`** with the following code. The code will fetch important state-sync information (such as block height and trust hash) from our server and update your **`config.toml`** file accordingly.

```shell
nano juno_state_sync.sh
```

```shell
#!/bin/bash

SNAP_RPC="https://juno-testnet-rpc.polkachu.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.juno/config/config.toml
```

Grant user the privilege to execute script and then run the script:

```shell
chmod 700 juno_state_sync.sh
./juno_state_sync.sh
```

Back up your wasm folder. Let's assume that your data location is at **`~/.juno`**:

```shell
cp -r ~/.juno/data/wasm ~
```

Reset the node. Notice that the following command will wipe off your data, including the wasm folder. So make sure that the last step is properly completed.

```shell
# On some tendermint chains
junod unsafe-reset-all

# On other tendermint chains
junod tendermint unsafe-reset-all --home $HOME/.juno
```

Copy the wasm folder back to its proper location:

```shell
cp -r ~/wasm ~/.juno/data
```

Restart the node:

```shell
sudo service cosmovisor start
```

If everything goes well, your node should start syncing within 10 minutes.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
junod tx staking create-validator \
  --amount 9000000ujuno \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(junod tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.0025ujunox \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
junod tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.juno/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
