---

Original guide can be found here:

[GitHub - gh0stdotexe/teritori-mainnet-genesis: Teritori mainnet genesis](https://github.com/gh0stdotexe/teritori-mainnet-genesis "GitHub - gh0stdotexe/teritori-mainnet-genesis: Teritori mainnet genesis")

Snapshot:

<br>

Block Explorer:

[Teritori explorer by Nodes.Guru](https://teritori.explorers.guru/validators "Teritori explorer by Nodes.Guru")

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
git clone https://github.com/TERITORI/teritori-chain teritori
cd teritori
git checkout v1.3.0
make install
```

To confirm that the installation has succeeded, you can run:

```shell
teritorid version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Configure Node

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

```shell
teritorid init YOUR_MONIKER --chain-id teritori-1
```

### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/teritori/genesis.json --inet4-only
mv genesis.json ~/.teritorid/config
```

## Set minimum gas prices

In `$HOME/.teritorid/config/app.toml`, set minimum gas prices:

```shell
sed -i.bak 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0utori"/' $HOME/.teritorid/config/app.toml
```

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.teritorid/config/config.toml`:

```shell
sed -i.bak ‘s/persistent_peers =.*/persistent_peers = “40caa979c29a9930ea2b8a6249037924d308ae84@162.55.234.70:54256,4ad852b6bc3fd9c2b5838a6f9c6d75380442c414@185.237.253.91:46657,7f9773971291b77b2d65364a8928cb31c40aa70f@65.108.73.124:13656,647bbbc30d26fbbb2f7d19aafe30ed77a92c4748@[2a01:4f9:6b:2e5b::4]:26656,36c2418b7aed4e585ac3e8f138a2e5ccf0f8278f@198.244.228.17:28766,a6d2c4de332606a98915e98e6a2d042779464680@161.97.155.94:26656,97838a0c8a5035398f696dd29f28fe66b20b6a8d@46.4.81.204:44656,719fec9bd14d52d5ea1048efa6d749e256811292@65.108.140.110:2665,81bd965baf90d49b9ff3e122394150fcdc935e64@peer.teritori.silknodes.io:26604,c669be4c7c0e44a3da941f4b97a8ee4ef39f7d6e@teritori.maethstro.com:26656,5bca681879abab92404f7b8f50a6ec5bd0b36d62@138.201.216.87:29656,a5d407fa1aec492dce5aff2e68a3b9c4d0503016@65.108.206.56:26656,629bc7199de18520f348021451e80082e8a0618e@13.214.246.107:26656,9efb86947ea9883aa82a4b9a39532546c9dbc499@95.217.36.166:26656,5ab6437f73fe71f392d53566e037aa91087530ac@139.144.67.202:26656,ed090020aba4bb254ba1517644ab0d6c94c9461e@57.128.144.230:26656,010f61b6cb13193853be4f42c328df3a0680d4d6@65.108.101.50:60656,6bc9f80a5123d62c23aadb7b5d68b740a794b0c6@65.109.49.111:36656,6ae90b7eedcf7b7712aba3e23c35d7b784144e34@peer.teritori.stavr.tech:21096,d856120f262134ebf13e1d2632d778b69e704208@65.108.4.188:15956,e73e8cefd738de437775f9621a8bd76f1e6ff954@18.191.35.171:26656,3069b058b5ed85c3cdb2cf18fb1d255d966b53af@193.149.187.8:26656,a06fbbb9ace823ae28a696a91daa2d0644653c28@65.21.32.200:26756,7fbfea037bd7962199ffbfd25986c014bab05298@65.108.140.17:32656”/’ $HOME/.teritorid/config/config.toml
```

## Addrbook for Teritori

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/teritori/addrbook.json --inet4-only
mv addrbook.json ~/.teritorid/config
```

## Create (or restore) a local key pair

Either create a new key pair, or restore an existing wallet for your validator:

```shell
# Create new keypair
teritorid keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
teritorid keys add WhisperNode --recover

# Query the keystore for your public address
teritorid keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
teritorid tx staking create-validator \
  --amount 1000000utori \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(teritorid tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --identity=""
  --fees 0utori \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
teritorid tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.teritorid/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
