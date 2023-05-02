---

Original Guide:

[empowerchain/testnets/altruistic-1 at main · empowerchain/empowerchain](https://github.com/empowerchain/empowerchain/tree/main/testnets/altruistic-1 "empowerchain/testnets/altruistic-1 at main · empowerchain/empowerchain")

Block Explorer:

[MantiCore Explorer](https://testnet.manticore.team/empower/staking/empowervaloper1qjg3cqx8w46drfyj96tsqwksle7cg5ywhts9y0 "MantiCore Explorer")

Polkachu Testnet Lab:

[Empower Testnet | Polkachu](https://polkachu.com/testnets/empower "Empower Testnet | Polkachu")

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

Unless you want to configure in a nonstandard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

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
cd $HOME && git clone https://github.com/empowerchain/empowerchain
cd empowerchain
git pull
git checkout v0.0.3
cd chain && make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
empowerd version
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=altruistic-1
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME` (using this random 32-bit hex hash generator: <https://numbergenerator.org/hex-code-generator#!numbers=1&length=32&addfilters=>)

```shell
MONIKER_NAME="32BITHEXHASH"
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
empowerd init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.empowerchain/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd $HOME/.empowerchain/config
curl -s https://raw.githubusercontent.com/empowerchain/empowerchain/main/testnets/altruistic-1/genesis.json > ~/.empowerchain/config/genesis.json
```

This will replace the genesis file created using `empowerd init` command with the mainnet `genesis.json`.

Check genesis Sha256sum:

```shell
sha256sum $HOME/.empowerchain/config/genesis.json

# should output 
fcae4a283488be14181fdc55f46705d9e11a32f8e3e8e25da5374914915d5ca8
```

## Set seeds + persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.empowerchain/config/config.toml`:

```shell
seeds=""
peers="9de92b545638f6baaa7d6d5109a1f7148f093db3@65.108.77.106:26656,4fd5e497563b2e09cfe6f857fb35bdae76c12582@65.108.206.56:26656,fe32c17373fbaa36d9fd86bc1146bfa125bb4f58@5.9.147.185:26656,220fb60b083bc4d443ce2a7a5363f4813dd4aef4@116.202.236.115:26656,225ad85c594d03942a026b90f4dab43f90230ea0@88.99.3.158:26656,2a2932e780a681ddf980594f7eacf5a33081edaf@192.168.147.43:26656,333de3fc2eba7eead24e0c5f53d665662b2ba001@10.132.0.11:26656,4a38efbae54fd1357329bd583186a68ccd6d85f9@94.130.212.252:26656,52450b21f346a4cf76334374c9d8012b2867b842@167.172.246.201:26656,56d05d4ae0e1440ad7c68e52cc841c424d59badd@192.168.1.46:26656,6a675d4f66bfe049321c3861bcfd19bd09fefbde@195.3.223.204:26656,1069820cdd9f5332503166b60dc686703b2dccc5@138.201.141.76:26656,277ff448eec6ec7fa665f68bdb1c9cb1a52ff597@159.69.110.238:26656,3335c9458105cf65546db0fb51b66f751eeb4906@5.189.129.30:26656,bfb56f4cb8361c49a2ac107251f92c0ea5a1c251@192.168.1.177:26656,edc9aa0bbf1fcd7433fcc3650e3f50ab0becc0b5@65.21.170.3:26656,d582bcd8a8f0a20c551098571727726bc75bae74@213.239.217.52:26656,eb182533a12d75fbae1ec32ef1f8fc6b6dd06601@65.109.28.219:26656,b22f0708c6f393bf79acc0a6ca23643fe7d58391@65.21.91.50:26656,e8f6d75ab37bf4f08c018f306416df1e138fd21c@95.217.135.41:26656,ed83872f2781b2bdb282fc2fd790527bcb6ffe9f@192.168.3.17:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.empowerchain/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
empowerd keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
empowerd keys add WhisperNode --recover

# Query the keystore for your public address
empowerd keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 05ce5993.md>)

---

## [Polkachu State-Sync Server Setup](https://polkachu.com/testnets/kujira/state_sync "Kujira Testnet Node State-Sync | Polkachu")

Our state-sync RPC server for Comdex is:

```shell
https://kujira-testnet-rpc.polkachu.com:443
```

## Instruction

We assume that you use Cosmovisor to manage your node. If you do not use Cosmovisor, you will need to customize the following instruction slightly.

Create a reusable shell script such as **`state_sync.sh`** with the following code. The code will fetch important state-sync information (such as block height and trust hash) from our server and update your **`config.toml`** file accordingly.

```shell
nano kujira_state_sync.sh
```

```shell
#!/bin/bash

SNAP_RPC="https://kujira-testnet-rpc.polkachu.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kujira/config/config.toml
```

Grant user the privilege to execute the script and then run the script:

```shell
chmod 700 kujira_state_sync.sh
./kujira_state_sync.sh
```

Stop the node:

```shell
sudo service cosmovisor stop
```

Reset the node:

```shell
# On some tendermint chains
kujirad unsafe-reset-all

# On other tendermint chains
kujirad tendermint unsafe-reset-all --home $HOME/.kujira
```

Restart the node:

```shell
sudo service cosmovisor start
```

If everything goes well, your node should start syncing within 10 minutes.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
kujirad tx staking create-validator \
  --amount 1000000ukuji \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(kujirad tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ukuji \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
kujirad tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.defund/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
