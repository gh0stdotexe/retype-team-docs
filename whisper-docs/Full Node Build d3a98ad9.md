---

**Original guide can be found on the Cosmos Github:**

[genesisTransactions/mantle-1 at main · AssetMantle/genesisTransactions](https://github.com/AssetMantle/genesisTransactions/tree/main/mantle-1#mantle-1 "genesisTransactions/mantle-1 at main · AssetMantle/genesisTransactions")

**Snapshots:**

[Assetmantle Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/assetmantle "Assetmantle Validator Node Snapshot | Polkachu")

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
source ~/.zshrc
```

# Build Daemon from source

```shell
git clone https://github.com/AssetMantle/node.git
cd node
git checkout v0.3.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
mantleNode version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=mantle-1
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

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
mantleNode init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.mantleNode/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd ~/.mantleNode/config
rm genesis.json
curl -fsSL https://raw.githubusercontent.com/AssetMantle/genesisTransactions/main/mantle-1/final_genesis.json >> genesis.json
```

This will replace the genesis file created using `mantleNode init` command with the mainnet `genesis.json`.

## Set persistent peers + Gas

Using `sed`, we can set the `persistent_peers` + `minimum-gas-prices` easily:

```shell
sed -i'' 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025umntl"/' $HOME/.mantleNode/config/app.toml
sed -i'' 's/persistent_peers = ""/persistent_peers = "6e08b23315a9f0e1b23c7ed847934f7d6f848c8b@165.232.156.86:26656,ee27245d88c632a556cf72cc7f3587380c09b469@45.79.249.253:26656,538ebe0086f0f5e9ca922dae0462cc87e22f0a50@34.122.34.67:26656,d3209b9f88eec64f10555a11ecbf797bb0fa29f4@34.125.169.233:26656,bdc2c3d410ca7731411b7e46a252012323fbbf37@34.83.209.166:26656,585794737e6b318957088e645e17c0669f3b11fc@54.160.123.34:26656,5b4ed476e01c49b23851258d867cc0cfc0c10e58@206.189.4.227:26656"/' $HOME/.mantleNode/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
mantleNode keys add WhisperNode

# Restore existing gaia wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
mantleNode keys add WhisperNode --recover

# Query the keystore for your public address
mantleNode keys show WhisperNode -a
```

==After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.==

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

## Addrbook for Asset Mantle

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots1.polkachu.com/addrbook/assetmantle/addrbook.json
mv addrbook.json ~/.mantleNode/config
```

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
mantleNode tx staking create-validator \
  --amount 1000000umntl \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(mantleNode tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id mantle-1 \
  --gas-prices 0.025umntl \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
mantleNode tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.`mantleNode`/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
