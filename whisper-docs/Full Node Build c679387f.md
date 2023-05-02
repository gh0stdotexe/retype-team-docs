---

*Original guide can be found on the ChihuahuaChain Github:*

[GitHub - ChihuahuaChain/mainnet](https://github.com/ChihuahuaChain/mainnet "GitHub - ChihuahuaChain/mainnet")

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

# Build Daemon from source

```shell
git clone https://github.com/ChihuahuaChain/chihuahua.git
cd chihuahua
git checkout v4.1.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
chihuahuad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=chihuahua-1
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

## Set minimum gas prices

In `$HOME/.chihuahua/config/app.toml`, set minimum gas prices:

```shell
nano ~/.chihuahua/config/app.toml

# update gas prices
minimum-gas-prices = "1uhuahua"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
chihuahuad init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.chihuahua/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O ~/.chihuahua/config/genesis.json https://raw.githubusercontent.com/ChihuahuaChain/chihuahua/main/mainnet/genesis.json
```

This will replace the genesis file created using `chihuahuad init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.chihuahua/config/config.toml`:

```shell
sed -i "s/persistent_peers =.*/persistent_peers = \"b140eb36b20f3d201936c4757d5a1dcbf03a42f1@216.238.79.138:26656,19900e1d2b10be9c6672dae7abd1827c8e1aad1e@161.97.96.253:26656,c382a9a0d4c0606d785d2c7c2673a0825f7c53b2@88.99.94.120:26656,a5dfb048e4ed5c3b7d246aea317ab302426b37a1@137.184.250.180:26656,3bad0326026ca4e29c64c8d206c90a968f38edbe@128.199.165.78:26656,89b576c3eb72a4f0c66dc0899bec7c21552ea2a5@23.88.7.73:29538,38547b7b6868f93af1664d9ab0e718949b8853ec@54.184.20.240:30758,a9640eb569620d1f7be018a9e1919b0357a18b8c@38.146.3.160:26656,7e2239a0d4a0176fe4daf7a3fecd15ac663a8eb6@144.91.126.23:26656\"/g" "${HOME}"/.chihuahua/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
chihuahuad keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
chihuahuad keys add WhisperNode --recover

# Query the keystore for your public address
chihuahuad keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

<br>

# Addrbook for Chihuahua

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots1.polkachu.com/addrbook/chihuahua/addrbook.json
mv addrbook.json ~/.chihuahua/config
```

# Syncing the node

After starting the junod daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block and you are safe to proceed to upgrade to a validator node.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
chihuahuad tx staking create-validator \
  --amount 1000000uhuahua \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \sudo -S systemctl daemon-reload 
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(chihuahuad tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uhuahua \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
chihuahuad tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.chihuahua/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
