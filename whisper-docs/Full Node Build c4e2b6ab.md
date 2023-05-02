---

Original Guide:

[network/testnets/stafihub-public-testnet-2 at main · stafihub/network](https://github.com/stafihub/network/tree/main/testnets/stafihub-public-testnet-2 "network/testnets/stafihub-public-testnet-2 at main · stafihub/network")

Block Explorer:

[StaFi Hub Explorer](https://testnet-explorer.stafihub.io/stafi-hub-testnet/staking "StaFi Hub Explorer")

<br>

---

# Install pre-requisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y

# install toolchain and ensure accurate time synchronization
sudo apt-get install make build-essential gcc git jq chrony -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
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
git clone --branch public-testnet-v3 https://github.com/stafihub/stafihub
cd $HOME/stafihub && make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
stafihubd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=stafihub-public-testnet-3
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
stafihubd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.stafihub/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-3/genesis.json"
```

This will replace the genesis file created using `stafihubd init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.stafihub/config/config.toml`:

```shell
peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```

## Update Min Gas Fees + Timeout Commit

```shell
# update gas prices
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml

# grpc
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
stafihubd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
stafihubd keys add WhisperNode --recover

# Query the keystore for your public address
stafihubd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File ee991f7b.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
stafihubd tx staking create-validator \
  --amount 1000000ufis \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(stafihubd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ufis \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
stafihubd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.stafihub/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
