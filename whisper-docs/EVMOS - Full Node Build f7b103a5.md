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

Unless you want to configure in a non-standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

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

# Build Daemon from source

```shell
git clone https://github.com/tharsis/evmos.git
cd evmos
git checkout v11.0.1
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
evmosd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
evmosd config chain-id evmos_9001-2
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
evmosd init WhisperNode --chain-id evmos_9001-2
```

This will generate the following files in `~/.evmosd/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

For Evmos genesis, we need to grab the new file and replace our existing one with it:

```shell
wget https://archive.evmos.org/mainnet/genesis.json
mv genesis.json ~/.evmosd/config/

# verify shasum

sha256sum $HOME/.evmosd/config/genesis.json

# should output
4aa13da5eb4b9705ae8a7c3e09d1c36b92d08247dad2a6ed1844d031fcfe296c
```

This will replace the genesis file created using `evmosd init` command with the mainnet `genesis.json`.

Next, we'll validate the `evmosd` genesis file:

```shell
evmosd validate-genesis
```

## Set seeds list

Update our p2p seeds found in `~/.evmosd/config/config.toml :`

```shell
# p2p seeds
seeds="eaa3dae2275faf9f599690c336d0e41e59fa6ae0@65.108.6.69:26656,4e5597c2153c1a5b56ecaccff0bd49340bbc1de2@65.108.137.35:26656,7aa31684d201f8ebc0b1e832d90d7490345d77ee@52.10.99.253:26656,5740e4a36e646e80cc5648daf5e983e5b5d8f265@54.39.18.27:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.evmosd/config/config.toml
```

## Addrbook for Evmos

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/evmos/addrbook.json
mv addrbook.json ~/.evmosd/config
```

## Update Gas + EVM Configuration:

```shell
# update gas
nano $HOME/.evmosd/config/app.toml
minimum-gas-prices = "25000000000aevmos"


# update EVM configuration

###############################################################################
###                             EVM Configuration                           ###
###############################################################################

[evm]

# Tracer defines the 'vm.Tracer' type that the EVM will use when the node is run in
# debug mode. To enable tracing use the '--evm.tracer' flag when starting your node.
# Valid types are: json|struct|access_list|markdown
tracer = ""

# MaxTxGasWanted defines the gas wanted for each eth tx returned in ante handler in check tx mode.
max-tx-gas-wanted = 500000
```

## Restore from Snapshot

[Evmos Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/evmos "Evmos Validator Node Snapshot | Polkachu")

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
evmosd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
evmosd keys add WhisperNode --recover

# Query the keystore for your public address
evmosd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

## Optional: Setup Cosmovisor

Follow the instructions here to setup Cosmovisor:

[Setup Cosmovisor](<Setup Cosmovisor 8f9695e5.md>)

---

## Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
evmosd tx staking create-validator
  --amount=1nano aevmos
  --pubkey=$(evmosd tendermint show-validator)
  --moniker="WhisperNode🤐"
  --chain-id=evmos_9001-2
  --identity="9C7571030BEF5157"
  --website="https://WhisperNode.com"
  --details="WhisperNode operates robust, high up-time validators across multiple Cosmos-based blockchains. We are highly active participants in the communities we validate for as we believe this is a necessity for effective on-chain governance and staying up to date on all network issues."
  --commission-rate="0.05"
  --commission-max-rate="0.20"
  --commission-max-change-rate="0.01"
  --min-self-delegation="1000000"
  --gas="auto"
  --gas-prices="0.025aevmos"
  --from=WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, keybase.io id, etc.

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.evmosd/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
