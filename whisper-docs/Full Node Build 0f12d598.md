---

Original Guide:

[testnets/genesis\_validators.md at main · dymensionxyz/testnets](https://github.com/dymensionxyz/testnets/blob/main/dymension-hub/35-C/genesis_validators.md#part-2 "testnets/genesis_validators.md at main · dymensionxyz/testnets")

Block Explorer:

<br>

Polkachu Testnet Support:&#x20;

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
git clone https://github.com/dymensionxyz/dymension.git --branch v0.2.0-beta
cd dymension
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
dymd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=35-C
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
dymd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.dymension/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/dymensionxyz/testnets/main/dymension-hub/35-C/genesis.json > ~/.dymension/config/genesis.json
sha256sum ~/.dymension/config/genesis.json
# should output:
# cf20e3b15d089ceeaaa9bb2abcd48a50f98e9f2274f4320aeae534d6972c4ee2
# validate genesis
dymd validate-genesis
```

This will replace the genesis file created using `dymd init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.dymension/config/config.toml`:

```shell
PEERS=281190aa44ca82fb47afe60ba1a8902bae469b2a@dymension.peers.stavr.tech:17806,e6ea3444ac85302c336000ac036f4d86b97b3d3e@38.146.3.199:20556,c17a4bcba59a0cbb10b91cd2cee0940c610d26ee@95.217.144.107:20556,b473a649e58b49bc62b557e94d35a2c8c0ee9375@95.214.53.46:36656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.dymension/config/config.toml
```

## Set Seeds

Using the peers variable we set earlier, we can set the  `seeds` in `~/.dymension/config/config.toml`:

```shell
SEEDS=b78dd0e25e28ec0b43412205f7c6780be8775b43@dym.seed.takeshi.team:10356,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@dymension-testnet.seed.brocha.in:30584,babc3f3f7804933265ec9c40ad94f4da8e9e0017@testnet-seed.rhinostake.com:20556
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" $HOME/.dymension/config/config.toml
```

##

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
dymd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
dymd keys add WhisperNode --recover

# Query the keystore for your public address
dymd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File abd19479.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
dymd tx staking create-validator \
  --amount 1000000udym \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(dymd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025udym \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
dymd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.dymension/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
