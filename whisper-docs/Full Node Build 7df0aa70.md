---

Original Guide:

[networks/Genesis.md at main · quasar-finance/networks](https://github.com/quasar-finance/networks/blob/main/docs/Genesis.md "networks/Genesis.md at main · quasar-finance/networks")

Block Explorer:

[Ping Dashboard](https://explorer.kjnodes.com/quasar "Ping Dashboard")

KJnodes Quasar Support:

[Quasar - kjnodes | Chain services](https://services.kjnodes.com/home/mainnet/quasar "Quasar - kjnodes | Chain services")

<br>

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
wget https://github.com/quasar-finance/quasar-preview/releases/download/v0.1.0/quasarnoded-linux-amd64?raw=true
mv 'quasarnoded-linux-amd64?raw=true' ~/go/bin/quasarnoded
sudo chmod +x ~/go/bin/quasarnoded
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
quasarnoded version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=quasar-1
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

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
quasarnoded init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.quasarnode/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/quasar-finance/networks/main/quasar-1/definitive-genesis.json > ~/.quasarnode/config/genesis.json
```

This will replace the genesis file created using `quasarnoded init` command with the mainnet `genesis.json`.

## Set Chain Parameters

We'll update things like persistent peers, minimum gas prices, seed nodes, etc:

```shell
PEERS=298e0e1faf8a5da43514cc2908d2908658e732a0@38.146.3.148:18256,49b72b4c79d589955a5004797b45ee306da6a889@143.42.237.237:26656,631775a6e058b5dfca4c68dfb3afec5126301c1b@51.75.146.179:8090,b417b1e3125ad895a280999f0749bca3f871bfd2@65.21.193.117:8090,771659b9205187f9094f894c65d29effa79fdd2c@18.156.191.84:26656,d9bfa29e0cf9c4ce0cc9c26d98e5d97228f93b0b@quasar.rpc.kjnodes.com:48656,5e02509b0aea383251559cb5ee36f4143b8f2d8d@20.7.139.229:26656,cd460ad2c3bc64e74a44f0796ac23cf4f4ed2a04@23.111.23.236:26656,e62ce06e60a986ed04d2e080876a41e3b57a5304@93.190.141.218:26656,5a111b281852be31838ecf1202e59981e618355e@89.116.31.95:18256,d11f867df7e498de0835e2d1b5bc34334c7337d1@65.109.31.114:2490,1369d544be2680e031b57f30a8d18cbe8b17a8ef@54.38.73.121:26656,240c09f5d91d2c252cf29faa1a88aebd563d2561@57.128.144.247:26656,f2e7f8af9e5f72bcde83a8bc0ca05aded6d51a5e@103.180.28.199:26656,a14a40a5c83d4226fa1f902a8f488fd828336ba4@15.235.115.156:10005,982e80ee53fedcb54a19d5f0dba154a0c1aedc2a@3.34.113.161:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.quasarnode/config/config.toml

sed -i -e "s|^pruning *=.*|pruning = \"custom\"|" $HOME/.quasarnode/config/app.toml
sed -i -e "s|^pruning-keep-recent *=.*|pruning-keep-recent = \"113\"|" $HOME/.quasarnode/config/app.toml
sed -i -e "s|^pruning-keep-every *=.*|pruning-keep-every = \"0\"|" $HOME/.quasarnode/config/app.toml
sed -i -e "s|^pruning-interval *=.*|pruning-interval = \"17\"|" $HOME/.quasarnode/config/app.toml
sed -i -e "s|^snapshot-interval *=.*|snapshot-interval = \"0\"|" $HOME/.quasarnode/config/app.toml
sed -i -e "s|^snapshot-keep-recent *=.*|snapshot-keep-recent = \"2\"|" $HOME/.quasarnode/config/app.toml
```

<br>

```shell
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.01ibc/0471F1C4E7AFD3F07702BEF6DC365268D64570F7C1FDC98EA6098DD6DE59817B,0.01ibc/FA0006F056DB6719B8C16C551FC392B62F5729978FC0B125AC9A432DBB2AA1A5,0.01ibc/FA7775734CC73176B7425910DE001A1D2AD9B6D9E93129A5D0750EAD13E4E63A\"|" $HOME/.quasarnode/config/app.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
quasarnoded keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
quasarnoded keys add WhisperNode --recover

# Query the keystore for your public address
quasarnoded keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 2fdce612.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
quasarnoded tx staking create-validator \
  --amount 1000000uqsr \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(quasarnoded tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uqsr \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
quasarnoded tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.quasarnode/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
