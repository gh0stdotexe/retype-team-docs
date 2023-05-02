---

Original Guide:

[Setting Up a New Node | Gitopia](https://docs.gitopia.com/setting-up-node "Setting Up a New Node | Gitopia")

Block Explorer:

[NodeStake Explorer](https://explorer.nodestake.top/gitopia-testnet/staking "NodeStake Explorer")

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
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm -rf go1.19.3.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

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

# Build Daemon from source / Install Gitopia

```shell
# install Gitopia first
curl https://get.gitopia.com | bash
sudo mv /tmp/tmpinstalldir/git-remote-gitopia /usr/local/bin/

# build from source
git clone -b v1.2.0 gitopia://gitopia/gitopia
cd gitopia && make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
gitopiad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=gitopia-janus-testnet-2
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
gitopiad init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.gitopia/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget https://server.gitopia.com/raw/gitopia/testnets/master/gitopia-janus-testnet-2/genesis.json.gz
gunzip genesis.json.gz
mv genesis.json $HOME/.gitopia/config/genesis.json

# verify genesis sha256sum
shasum -a 256 $HOME/.gitopia/config/genesis.json

# should output
038a81d821f3d8f99e782cbfed609e4853d24843c48a1469287528e632a26162
```

This will replace the genesis file created using `gitopiad init` command with the mainnet `genesis.json`.

## Validate Genesis

```shell
gitopiad validate-genesis
```

## Set seeds

Using the peers variable we set earlier, we can set the `seeds` in `~/.gitopia/config/config.toml`:

```shell
sed -i 's#seeds = ""#seeds = "399d4e19186577b04c23296c4f7ecc53e61080cb@seed.gitopia.com:26656"#' $HOME/.gitopia/config/config.toml
```

## Update Min Gas Fees

```shell
# update gas prices

nano ~/.gitopia/config/app.toml
minimum-gas-prices = "0.001utlore"
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
gitopiad keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
gitopiad keys add WhisperNode --recover

# Query the keystore for your public address
gitopiad keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 3cf0c1ad.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
gitopiad tx staking edit-validator
  --moniker="choose a moniker" \
  --website="https://gitopia.com" \
  --identity=6A0D65E29A4CBC8B \
  --details="Code Collaboration for Web3" \
  --chain-id=<chain_id> \
  --gas="auto" \
  --gas-prices="0.001utlore" \
  --from=<key_name> \
  --commission-rate="0.10"
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, keybase.io id, etc. To see a full list:

```shell
gitopiad tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.gitopia/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
