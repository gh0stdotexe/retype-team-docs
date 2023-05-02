---

Original guide:

[Joining Mainnet | Osmosis Docs](https://docs.osmosis.zone/developing/network/join-mainnet.html#set-up-cosmovisor "Joining Mainnet | Osmosis Docs")

Github Link:

[GitHub - osmosis-labs/osmosis: The AMM Laboratory](https://github.com/osmosis-labs/osmosis "GitHub - osmosis-labs/osmosis: The AMM Laboratory")

Snapshot Link:

[Osmosis Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/osmosis "Osmosis Validator Node Snapshot | Polkachu")

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
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
rm -rf go1.19.5.linux-amd64.tar.gz
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
git clone https://github.com/osmosis-labs/osmosis
cd osmosis
git fetch
git checkout v14.0.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
osmosisd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=osmosis-1
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
MONIKER_NAME="WhisperNode-1"
```

## Set minimum gas prices

In `$HOME/.osmosisd/config/app.toml`, set minimum gas prices:

```shell
minimum-gas-prices = "0.025uosmo"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
osmosisd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.osmosisd/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O ~/.osmosisd/config/genesis.json https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json
```

## Set seeds list

Update our p2p seeds found in `~/.osmosisd/config/config.toml :`

```shell
# p2p seeds
63aba59a7da5197c0fbcdc13e760d7561791dca8@162.55.132.230:2000,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656
```

## Add Mempool Config to app.toml

```shell
### Goes at the bottom of app.toml ###


###############################################################################
###                      Osmosis Mempool Configuration                      ###
###############################################################################

[osmosis-mempool]
# This is the max allowed gas any tx. 
# This is only for local mempool purposes, and thus     is only ran on check tx.
max-gas-wanted-per-tx = "25000000"

# This is the minimum gas fee any arbitrage tx should have, denominated in uosmo per gas
# Default value of ".005" then means that a tx with 1 million gas costs (.005 uosmo/gas) * 1_000_000 gas = .005 osmo
arbitrage-min-gas-fee = ".005"

# This is the minimum gas fee any tx with high gas demand should have, denominated in uosmo per gas
# Default value of ".0025" then means that a tx with 1 million gas costs (.0025 uosmo/gas) * 1_000_000 gas = .0025 osmo
min-gas-price-for-high-gas-tx = ".0025"
```

# Setup Cosmovisor

Follow the [Setup Cosmovisor](<Setup Cosmovisor 49b881a7.md>) guide to set up Cosmovisor and start the node.

---

## Download Chain Data

Download the latest chain data from a snapshot provider. In the following commands, I will use [cosmos-snapshots/Osmosis.md at main · c29r3/cosmos-snapshots](https://github.com/c29r3/cosmos-snapshots/blob/main/Osmosis.md "cosmos-snapshots/Osmosis.md at main · c29r3/cosmos-snapshots") to download the chain data. You may choose the default, pruned, or archive based on your needs.

Delete any existing chain data (if any):

```shell
rm -rf ~/.osmosisd/data; \
mkdir -p ~/.osmosisd/data; \
cd ~/.osmosisd/data
```

```shell
# download snapshot
SNAP_NAME=$(curl -s http://135.181.60.250:5888/osmosis/ | egrep -o ">osmosis.*tar" | tr -d ">")
wget -O - http://135.181.60.250:5888/osmosis/${SNAP_NAME} | tar xf -
```

We can now restart Cosmovisor and start the chain:

```shell
sudo systemctl start cosmovisor
```

---

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
osmosisd keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
osmosisd keys add <key-name> --recover

# Query the keystore for your public address
osmosisd keys show <key-name> -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
osmosisd tx staking create-validator \
  --amount 9000000uosmo \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(osmosisd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uosmo \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
osmosisd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.osmosisd/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
