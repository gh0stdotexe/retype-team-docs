---

*Original Guide:*

[Join Mainnet | Injective Documentation](https://docs.injective.network/nodes/RunNode/mainnet "Join Mainnet | Injective Documentation")

*Snapshot links:*

[Injective Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/injective "Injective Node Snapshot | Polkachu")

Injective Integration Notion:

[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://injective.notion.site/Injective-Integration-Wiki-51f27c594c5c4f0eaae33f4aa6ddc51e "Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.")

Node Operators Wiki:

[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://injective.notion.site/Node-Operators-297159169099475eb2613b094fbee225 "Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.")

Peggo Monitoring Script:

[peggo/monitor at master · InjectiveLabs/peggo](https://github.com/InjectiveLabs/peggo/tree/master/monitor "peggo/monitor at master · InjectiveLabs/peggo")

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
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm -rf go1.20.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.zshrc
```

Add the "export Pathing" rules at the bottom, and then save the file:

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
wget https://github.com/InjectiveLabs/injective-chain-releases/releases/download/v1.10-1678709842/linux-amd64.zip
unzip linux-amd64.zip
sudo mv peggo /usr/bin
sudo mv injectived /usr/bin
sudo mv libwasmvm.x86_64.so /usr/lib 
```

To confirm that the installation has succeeded, you can run:

```shell
injectived version

# should output the following
Version dev (e218afcf7)
Compiled at 20230313-1224 using Go go1.18.3 (amd64)
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

We can generate a 32-bit moniker using the following command:

```shell
openssl rand -hex 16
```

```shell
injectived init <HEX> --chain-id injective-1
```

## Prepare configuration to join Mainnet

You should now update the default configuration with the Mainnet's genesis file and application config file, as well as configure your persistent peers with a seed node.

```shell
git clone https://github.com/InjectiveLabs/mainnet-config

# copy genesis file to config directory
cp mainnet-config/10001/genesis.json ~/.injectived/config/genesis.json

# copy config file to config directory
cp mainnet-config/10001/app.toml  ~/.injectived/config/app.toml
```

You can also run verify the checksum of the genesis checksum - `573b89727e42b41d43156cd6605c0c8ad4a1ce16d9aad1e1604b02864015d528`

```shell
sha256sum ~/.injectived/config/genesis.json
```

Then open update the persistent\_peers field present in `~/.injectived/config/config.toml` with the contents of `mainnet-config/10001/seeds.txt` and update the `timeout_commit` to `300ms`.

```shell
cat mainnet-config/10001/seeds.txt
nano ~/.injectived/config/config.toml
```

Persistent Peers

We can add these `persistent_peers` to our `config.toml`:

```shell
sed -i "s/persistent_peers =.*/persistent_peers = \"2ee7586d8e5a4a2020dafddf68efdfdaa3606cc3@mainnet-sentry-us-0.injective.dev:26656,30e81e1b785eaf82d2260e932251c02c95787740@mainnet-sentry-us-1.injective.dev:26656,de25916e19083d44d67e0c336199b215523f1578@mainnet-sentry-eu-0.injective.dev:26656,1a2ccd9c436eae54d6c767ad566c58dce78f8be2@mainnet-sentry-eu-1.injective.dev:26656,8c39e67ea72458c5192c4653837e7c172a031a62@mainnet-sentry-eu-2.injective.dev:26656,9842eedb736a7a9bbdca4b1be2a5cdb14224a12a@mainnet-sentry-eu-3.injective.dev:26656,7bb20f8e9fa73e34d0d21862528d5fba839eeb56@mainnet-sentry-asia-0.injective.dev:26656,2ab91484b1504090b088465dc650d44311362bfa@mainnet-sentry-asia-1.injective.dev:26656\"/g" "${HOME}"/.injectived/config/config.toml
```

## Addrbook for Injective

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/injective/addrbook.json --inet4-only
mv addrbook.json ~/.injectived/config
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
injectived keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
injectived keys add WhisperNode --recover

# Query the keystore for your public address
injectived keys show WhisperNode -a
```

> **After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

Skip.money - Sidecar Setup

```shell
# EXAMPLE below (please use the correct values)
[sidecar]
api_key = "N4kN71cF2HQce243xD+k876TXhY="
sentinel_peer_string = "6f3b548716049d83ab701a1eddef56bd202c09db@injective-1-sentinel.skip.money:26656"
sentinel_rpc_string = "https://injective-1-api.skip.money"
personal_peer_ids = ""
```

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](https://app.nuclino.com/t/b/cc174b4c-761a-4836-9f73-48ce977317d1) instructions to setup cosmovisor and start the node.

---

# Download Chain Data

Download the latest chain data from a snapshot provider. In the following commands, I will use [Injective Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/injective "Injective Node Snapshot | Polkachu") to download the chain data.

## How To Process Stargaze Snapshot

Install `lz4` if needed

```shell
sudo apt update
sudo apt install snapd -y
sudo snap install lz4
```

Download the snapshot

```shell
wget -O injective_28119508.tar.lz4 https://snapshots.polkachu.com/snapshots/injective/injective_28119508.tar.lz4 --inet4-only
```

Stop your node

```shell
sudo systemctl stop injectived
```

Reset your node. **WARNING**: This will erase your node database. If you are already running validator, be sure you backed up your **`priv_validator_key.json`** prior to running the command. The command does not wipe the file. However, you should have a backup of it already in a safe location.

```shell
injectived tendermint unsafe-reset-all --home $HOME/.injectived --keep-addr-book
```

Decompress the snapshot to your database location. Your database location will be something to the effect of **`~/.injectived`** depending on your node implemention.

```shell
lz4 -c -d injective_28119508.tar.lz4  | tar -x -C $HOME/.injectived
```

If everything is good, now restart your node

```shell
sudo service injectived start
```

---

# Upgrade to a validator

> **Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.**

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
injectived tx staking create-validator \
  --amount 1000000ustars \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(starsd tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ustars \
  --from <key-name>
```

> The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
starsd tx staking create-validator --help
```

<br>

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.starsd/config/`:

- `priv_validator_key.json`
- `node_key.json`

> It is recommended that you encrypt the backup of these files.

<br>
