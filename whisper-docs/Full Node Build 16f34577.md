---

*Original guide can be found on the Stargaze Github:*

<https://github.com/public-awesome/mainnet>

*Snapshot links:*

[Stargaze Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/stargaze "Stargaze Validator Node Snapshot | Polkachu")

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
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm -rf go1.19.3.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.zshrc
```

Add the "export Pathing" rules at the bottom, and then save the file:

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
git clone https://github.com/public-awesome/stargaze
cd stargaze
git checkout v7.5.0
make install
```

To confirm that the installation has succeeded, you can run:

```shell
starsd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

```shell
starsd init YOUR_MONIKER --chain-id stargaze-1
```

### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/stargaze/genesis.json --inet4-only
mv genesis.json ~/.starsd/config
```

## Set minimum gas prices

In `$HOME/.starsd/config/app.toml`, set minimum gas prices:

```shell
nano ~/.starsd/config/app.toml

# update gas prices
minimum-gas-prices = "0.025ustars"
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
starsd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.starsd/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl -s  https://raw.githubusercontent.com/public-awesome/mainnet/main/stargaze-1/genesis.tar.gz > genesis.tar.gz; tar -C ~/.starsd/config/ -xvf genesis.tar.gz
```

This will replace the genesis file created using `starsd init` command with the mainnet `genesis.json`.

## Addrbook for Stargaze

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/stargaze/addrbook.json --inet4-only
mv addrbook.json ~/.starsd/config
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
starsd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
starsd keys add WhisperNode --recover

# Query the keystore for your public address
starsd keys show WhisperNode -a
```

> **After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](https://app.nuclino.com/t/b/cc174b4c-761a-4836-9f73-48ce977317d1) instructions to setup cosmovisor and start the node.

---

# Download Chain Data

Download the latest chain data from a snapshot provider. In the following commands, I will use [Stargaze Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/stargaze "Stargaze Node Snapshot | Polkachu") to download the chain data.

## How To Process Stargaze Snapshot

Install lz4 if needed

```shell
sudo apt update
sudo apt install snapd -y
sudo snap install lz4
```

Download the snapshot

```shell
wget -O stargaze_5879446.tar.lz4 https://snapshots.polkachu.com/snapshots/stargaze/stargaze_5879446.tar.lz4 --inet4-only
```

Stop your node

```shell
sudo systemctl stop cosmovisor
```

Reset your node. **WARNING**: This will erase your node database. If you are already running validator, be sure you backed up your **`priv_validator_key.json`** prior to running the command. The command does not wipe the file. However, you should have a backup of it already in a safe location.

```shell
starsd tendermint unsafe-reset-all --home $HOME/.starsd --keep-addr-book
```

Decompress the snapshot to your database location. Your database location will be something to the effect of **`~/.starsd`** depending on your node implemention.

```shell
lz4 -c -d stargaze_5879446.tar.lz4  | tar -x -C $HOME/.starsd
```

If everything is good, now restart your node

```shell
sudo service stargaze start
```

---

# Upgrade to a validator

> **Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.**

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
starsd tx staking create-validator \
  --amount 1000000ustars \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(starsd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ustars \
  --from <key-name>
```

> The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
starsd tx staking create-validator --help
```

<br>

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.starsd/config/`:

- `priv_validator_key.json`
- `node_key.json`

> It is recommended that you encrypt the backup of these files.

<br>
