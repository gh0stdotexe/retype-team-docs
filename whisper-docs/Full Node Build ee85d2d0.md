---

Original Guide:

[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://docs-nolus-protocol.notion.site/Run-a-Full-Node-7a92545223e7483bb4a02cce30b53aa8 "Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.")

[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://docs-nolus-protocol.notion.site/Run-a-Validator-3b2657bc68ca4eb3a24078a2ccbb7680 "Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.")

Block Explorer:

<https://explorer-rila.nolus.io/nolus-rila/staking>

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
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.1.39
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
nolusd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=nolus-rila
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME="WhisperNode-0"
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
nolusd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.nolus/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json > ~/.nolus/config/genesis.json
```

This will replace the genesis file created using `nolusd init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.nolus/config/config.toml`:

```shell
PEERS="$(curl -s "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/persistent_peers.txt")"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.nolus/config/config.toml
```

**3.1 Adjust the minimum gas price:**

```shell
nano ~/.nolus/config/app.toml
minimum-gas-prices = "0.0025unls”
```

Modify the minimum-gas-prices field to set a minimum gas price threshold that your validator would be willing to accept in order to prevent spam attacks:

## Download Addrbook

Download the latest `addrbook.json` and replace our existing one to ensure that our node has the latest peers:

```shell
wget -O ~/.nolus/config/addrbook.json https://share2.utsa.tech/nolus/addrbook.json
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
nolusd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
nolusd keys add WhisperNode --recover

# Query the keystore for your public address
nolusd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor](<Setup Cosmovisor 180f19a6.md>)

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.sei/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
