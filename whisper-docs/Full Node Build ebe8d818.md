---

Original Guide:

[pylons-testnet/Pylons-testnet-3.md at main · Pylons-tech/pylons-testnet](https://github.com/Pylons-tech/pylons-testnet/blob/main/Pylons-testnet-3.md "pylons-testnet/Pylons-testnet-3.md at main · Pylons-tech/pylons-testnet")

Block Explorer:

[Pylons explorer by Nodes.Guru](https://pylons.explorers.guru/validators "Pylons explorer by Nodes.Guru")

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
git clone https://github.com/Pylons-tech/pylons
cd pylons
git checkout v0.4.2
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
pylonsd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=pylons-testnet-3
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
pylonsd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.pylons/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd $HOME/.pylons/config
curl -s https://raw.githubusercontent.com/Pylons-tech/pylons-testnet/main/genesis.json > ~/.pylons/config/genesis.json
```

This will replace the genesis file created using `pylonsd init` command with the mainnet `genesis.json`.

Next, we'll validate the genesis file to ensure it's correct:

```shell
pylonsd validate-genesis
```

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.pylons/config/config.toml`:

```shell
sed -i "s/persistent_peers =.*/persistent_peers = \"25e7ef64b41a636e3fb4e9bb1191b785e7d1d5cc@46.166.140.172:26656,2c50b8171af784f1dca3d37d5dda5e90f1e1add8@95.214.55.4:26656,4f90babf520599ffe606157b0151c4c9bc0ec23f@194.163.172.115:26666,ebecc93e7865036fbdf8d3d54a624941d6e41ba1@104.200.136.57:26656,022ee5a5231a5dec014841394f8ce766d657cff5@95.214.53.132:26156,a6972be573807d34f28a337c0f7d599e0014be80@161.97.99.247:26656,515ffd755a92a47b56233143f7c25481dbe99f94@161.97.99.251:26606\"/g" "${HOME}"/.pylons/config/config.toml
```

## Update Min Gas Fees + Timeout Commit

```shell
# update gas prices

nano ~/.pylons/config/app.toml
minimum-gas-prices = "0.025ubedrock"
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
pylonsd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
pylonsd keys add WhisperNode --recover

# Query the keystore for your public address
pylonsd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File a732ce4a.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
pylonsd tx staking create-validator \
  --amount 1000000ubedrock \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(pylonsd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ubedrock \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
pylonsd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.pylons/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
