---

Original Install Guide:\
[Installation - Sentinel Docs](https://docs.sentinel.co/sentinelhub/installation/ "Installation - Sentinel Docs")

Snapshot Repo:\
[cosmos-snapshots/Sentinel.md at main · c29r3/cosmos-snapshots](https://github.com/c29r3/cosmos-snapshots/blob/main/Sentinel.md "cosmos-snapshots/Sentinel.md at main · c29r3/cosmos-snapshots")

# Install pre-requisites&#x20;

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
git clone https://github.com/sentinel-official/hub.git 
cd hub
git checkout v0.10.1
make install
```

To confirm that the installation has succeeded, you can run:

```shell
sentinelhub version --long
```

Commit hash must be `fa7cd3c7d5f427308d8a837a18b951482ce5c9e2.`

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=sentinelhub-2
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

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
sentinelhub init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.sentinelhub/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
# download Genesis file
curl -s http://cosmosia12.notional.ventures:11111/sentinel/data/genesis.json > ~/.sentinelhub/config/genesis.json
# install unzip
sudo apt-get install unzip

# extract / select 'Y' option when asked to overwrite
unzip ${HOME}/genesis.zip -d ${HOME}/.sentinelhub/config/
```

This will replace the genesis file created using `sentinelhub init` command with the mainnet `genesis.json`.

## Set persistent peers

Persistent peers will be required to tell your node where to connect to other nodes and join the network. To retrieve the peers for the chosen mainnet:

```shell
# Set the base repo URL for mainnet & retrieve peers
persistent-peers = "05fe2a7847fd27345250915fd06752c424f40651@85.222.234.135:26656,387027e3b1180d3a619cbbf3462704a490785963@54.176.90.228:26656,63bd9cfce0f0d274aad5b166dd06d829021aec43@121.78.247.243:56656,855807cc6a919c22ec943050ebb5c80b23724ed0@3.239.11.246:26656,8caefbf8f4318ecc93f2c901cf11470e4a16c818@161.97.135.122:26656,9174af5f16f74660cccf49f893d243949af45f7f@54.177.29.46:26656,9fa528bd2b9e7c80724a1d8a4e1a2a8a83e7d123@142.93.72.221:26656,a77f6a094578dad899e2f40e0626b4c6d4705311@3.36.165.232:26656,bd45a11390d16d128a9eeea3935b53d7a1a3c120@15.236.127.69:26656,cdb8dd7628460a546ce1594ca0bc0c20366514cf@34.72.64.178:26656,d1efceccb04ded9a604e5235f76da86872157d68@161.97.149.223:26656,e00b23444cc8dbb353d5faa765ab36cfc0116b57@83.60.98.134:28685,e5ee89bd4fc371c6a0e66d2b8daefd891b6b87b5@157.90.117.58:26656,f7ceb735606f90df7eb6cd987641876955b6e325@46.4.55.150:36656"
```

NB: If you are unsure about this, you can ask in discord for the current peers and explicitly set them in `~/.sentinelhub/config/config.toml` instead.

## Set minimum gas prices

In `$HOME/.sentinelhub/config/app.toml`, set minimum gas prices:

```shell
minimum-gas-prices = "0.01udvpn,0.01ibc/A8C2D23A1E6F95DA4E48BA349667E322BD7A6C996D8A4AAE8BA72E190F3D1477,0.01ibc/ED07A3391A112B175915CD8FAF43A2DA8E4790EDE12566649D0C2F97716B8518"
```

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to set up cosmovisor and start the node.

---

## Download Snapshot

After starting the `digd` daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```shell
# Download and export snapshot (look up link to latest snapshot)
wget http://cosmosia12.notional.ventures:11111/sentinel/data_20221205_202255.tar.gz
tar -xvzf data_20221205_202255.tar.gz
mv data ~/.sentinelhub

# download Addrbook for fast peers connection
curl -s http://cosmosia12.notional.ventures:11111/sentinel/addrbook.json > ~/.sentinelhub/config/addrbook.json
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block, and you are safe to proceed to upgrade to a validator node.

---

### OPTIONAL: Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
sentinelhub keys add WhisperNode

# Restore existing dig wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
sentinelhub keys add WhisperNode --recover

# Query the keystore for your public address
sentinelhub keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

# Syncing the node

After starting the `digd` daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain but could take a very long time. To query the status of your node:

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
sentinelhub tx staking create-validator \
  --amount 1000000udig \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(sentinelhub tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025udvpn \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
sentinelhub tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.sentinelhub/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
