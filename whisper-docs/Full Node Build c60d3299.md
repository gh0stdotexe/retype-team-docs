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
wget https://golang.org/dl/go1.18.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz
rm -rf go1.18.4.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.profile 

# zsh
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
source ~/.profile

# ZSH
source ~/.zshrc
```

# Build Daemon from source

```shell
git clone https://github.com/comdex-official/comdex.git
cd comdex
git fetch --tags
git checkout v1.0.0
make all
```

To confirm that the installation has succeeded, you can run:

```shell
comdex version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=comets-test
```

Then source it:

```shell
source ~/.profile

# if using ZSH
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

In `$HOME/.comdex/config/app.toml`, set minimum gas prices:

```shell
nano ~/.comdex/config/app.toml
minimum-gas-prices = "0.2ucmdx"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
comdex init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.comdex/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd $HOME/.comdex/config
curl https://raw.githubusercontent.com/comdex-official/networks/main/testnet/comets-test/genesis_final.json > ~/.comdex/config/genesis.json
```

This will replace the genesis file created using `comdex init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.comdex/config/config.toml`:

```shell
# persistent peers list
PEERS=9c25a7ab94a315f683c3693e17aec6b2c91c851c@52.77.115.73:26656,8b1ccf5cf3a3ba65ee074f46ea8c6c164d867104@52.201.166.91:26656,5307ce50bd8a6f7bb5a922e3f7109b5f3241c425@13.51.118.56:26656,3659590cd1466671a49421089e55f1392e1cad0e@15.207.189.210:26656,295a24ae280ab9323bbd40036d9334926136adaf@108.61.99.116:26656,a26081c37215d11f2513f9a93e0925a3e3d3f110@44.200.185.103:26656,73c2b5e49bac8cefd7cbb89f75cf7c745c449a19@65.108.66.4:10056,e290237daeb22cce48eb25a5b468f80d3dc547e6@65.108.103.236:22656,5f9fbda6543fdf0f0240ca6099c9d01e58ce6a07@141.94.170.26:24456,f34396042110717912b2d221d0b703787f56a17e@23.88.64.161:26656,865a47c33b0115cee680826dacac8b191a630d89@142.132.230.221:26656,e3da7120edfc0f7e50e562153b485753300651df@173.249.40.87:36656,1887b6517973bfa5ae0d89fedbef7215072064fe@94.250.203.3:46656,04336b23169a4d871c6e4d3819444183eb285064@162.55.83.146:26656,e1d55b98e4fcde635694206f86384c70d74b4799@144.91.101.46:36396
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.comdex/config/config.toml
```

---

## [Polkachu State-Sync Peer for Comdex Testnet](https://polkachu.com/testnets/comdex/peers "Comdex Testnet Node Peers | Polkachu")

When you state-sync, you might also consider adding Polkachu's state-sync peer to your **`persistent_peers`** setting in **`config.toml`**.

```shell
6130e02dc9d7f30f8d4ab73c396c2197585e2c8c@65.108.225.158:31656
```

<br>

## Create (or restore) a local key pair

Either create a new key pair, or restore an existing wallet for your validator:

```shell
# Create new keypair
comdex keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
comdex keys add <key-name> --recover

# Query the keystore for your public address
comdex keys show <key-name> -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](https://app.nuclino.com/t/b/d8e17147-37cc-4043-9548-ff0e1ae4584e)

---

## [Polkachu State-Sync Server Setup](https://polkachu.com/testnets/comdex/state_sync "Comdex Testnet Node State-Sync | Polkachu")

Our state-sync RPC server for Comdex is:

```shell
https://comdex-testnet-rpc.polkachu.com:443
```

## Instruction

We assume that you use Cosmovisor to manage your node. If you do not use Cosmovisor, you will need to customize the following instruction slightly.

Create a reusable shell script such as **`state_sync.sh`** with the following code. The code will fetch important state-sync information (such as block height and trust hash) from our server and update your **`config.toml`** file accordingly.

```shell
nano comdex_state_sync.sh
```

```shell
#!/bin/bash

SNAP_RPC="https://comdex-testnet-rpc.polkachu.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.comdex/config/config.toml
```

Grant user the privilege to execute script and then run the script:

```shell
chmod 700 comdex_state_sync.sh
./comdex_state_sync.sh
```

Stop the node:

```shell
sudo service cosmovisor stop
```

Reset the node:

```shell
# On some tendermint chains
comdex unsafe-reset-all

# On other tendermint chains
comdex tendermint unsafe-reset-all --home $HOME/.comdex
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
comdex tx staking create-validator \
  --amount 9000000ujuno \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(comdex tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.2ucmdx \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
comdex tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.comdex/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
