---

Original Guide:

[GitHub - ingenuity-build/testnets: Quicksilver Testnet Instructions and Config](https://github.com/ingenuity-build/testnets "GitHub - ingenuity-build/testnets: Quicksilver Testnet Instructions and Config")

Block Explorer:

[Quicksilver explorer by Nodes.Guru](https://quicksilver.explorers.guru/validators "Quicksilver explorer by Nodes.Guru")

<br>

---

# Install pre-requisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y

# install toolchain and ensure accurate time synchronization
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
wget https://golang.org/dl/go1.18.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz
rm -rf go1.18.3.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.profile

# ZSH
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
git clone --branch v0.4.0 https://github.com/ingenuity-build/quicksilver.git
cd quicksilver && make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
icad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=kqcosmos-1
```

Then source it:

```shell
source ~/.profile

# ZSH
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME=<moniker-name>

# Example
MONIKER_NAME="WhisperNode-0"
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
icad init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.ica/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd ~/.ica/config
rm -rf genesis.json
https://raw.githubusercontent.com/ingenuity-build/testnets/main/killerqueen/kqcosmos-1/genesis.json
```

This will replace the genesis file created using `icad init` command with the mainnet `genesis.json`.

## Set seeds

Using the peers variable we set earlier, we can set the `seeds` in `~/.quicksilverd/config/config.toml`:

```shell
SEEDS="66b0c16486bcc7591f2c3f0e5164d376d06ee0d0@65.108.203.151:26656"
sed -i -e "/seeds =/ s/= .*/= "$SEEDS"/"  $HOME/.ica/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
icad keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
icad keys add WhisperNode --recover

# Query the keystore for your public address
icad keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 9e1f1e48.md>)

---

## [Polkachu State-Sync Server Setup](https://polkachu.com/testnets/quicksilver/state_sync "Quicksilver Testnet Node State-Sync | Polkachu")

Our state-sync RPC server for Quicksilver is:

```shell
https://quicksilver-testnet-rpc.polkachu.com:443
```

## Instruction

We assume that you use Cosmovisor to manage your node. If you do not use Cosmovisor, you will need to customize the following instruction slightly.

Create a reusable shell script such as **`state_sync.sh`** with the following code. The code will fetch important state-sync information (such as block height and trust hash) from our server and update your **`config.toml`** file accordingly.

```shell
nano quicksilver_state_sync.sh
```

```shell
#!/bin/bash

SNAP_RPC="https://quicksilver-testnet-rpc.polkachu.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.quicksilverd/config/config.toml
```

Grant user the privilege to execute the script and then run the script:

```shell
chmod 700 quicksilver_state_sync.sh
./quicksilver_state_sync.sh
```

Stop the node:

```shell
sudo service cosmovisor stop
```

Reset the node:

```shell
# On some tendermint chains
quicksilverd unsafe-reset-all

# On other tendermint chains
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd
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
quicksilverd tx staking create-validator \
  --amount 1000000uqck \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(quicksilverd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uqck \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
quicksilverd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.quicksilverd/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
