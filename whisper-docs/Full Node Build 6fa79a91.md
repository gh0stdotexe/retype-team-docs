---

Original Guide:

[Run a Node - Kujira Docs](https://docs.kujira.app/run-a-node "Run a Node - Kujira Docs")

Block Explorer:

[Kujira explorer by Nodes.Guru](https://kujira.explorers.guru/validators "Kujira explorer by Nodes.Guru")

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
wget https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
rm -rf go1.19.1.linux-amd64.tar.gz
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
git clone https://github.com/Team-Kujira/core $HOME/kujira-core
cd $HOME/kujira-core
git checkout v0.6.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
kujirad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=harpoon-4
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
kujirad init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.kujira/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
cd $HOME/.kujira/config
curl -s https://raw.githubusercontent.com/Team-Kujira/networks/master/testnet/harpoon-4.json > ~/.kujira/config/genesis.json
```

This will replace the genesis file created using `kujirad init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.kujira/config/config.toml`:

```shell
PEERS=8a07220e9094c83282b90b5835e1280f43891fd9@65.21.244.93:26656,e855a6d8063954d3c444f8c2848d019f438d9fae@209.145.59.96:26656,9e0441dbe67ff644f3e86da1a40b27f8c5edc7f9@88.227.70.178:26656,ccd2861990a98dc6b3787451485b2213dd3805fa@185.144.99.234:26656,0e2e52412e3a8887ad2950dcae11c298daf0745d@65.108.201.154:2060,0b4d4dae6953d954fc94362997dff3456764b2b4@5.9.138.91:26656,e3b81acf2eb2f071bc0d76a574a1205177df2266@165.22.108.35:26656,362468ed96561770cd32fc0f7cbd45d2e5985d38@146.19.24.243:26656,1a80d041e13cb211391785cfdec25fc8ce6fb4c4@157.90.183.167:26656,b5ccbdbd1d9a3bb7f7fa50279cacd78c023d5548@5.9.98.56:23095,d25c4cec6195e48176a2694bc3e17a6189de17e7@83.149.102.56:23095,79cda60f352601867efdbbbea72072a6241baa0f@95.216.7.241:40002,7616f1dd407daa09ca7956dab0f77de98d218682@145.239.70.8:36346,8a5b9e05aaa0203fc00dee3888a0a3955fc9ed3c@167.235.237.28:26656,20f0409359fa455d7ca11b81039782a260899579@195.201.164.223:26656,0bbe1fdcabb59e381fe0e461b04a1822fb1d656a@65.108.232.150:23095,5552e4eb91dd7fe15af0dbfcbc8dad002e46d69e@188.40.140.51:40001,ccb32ae7e8976a64a340f26610c54a3d4e8a9863@178.62.199.143:26656,3d207046e1410f67852908b8a11585d463448f11@135.181.157.37:26656,82273f9136166a332337faf3262d39acf6e0958b@194.163.164.52:26656,0136051cf3d9169b081f6d6771e2aa59a3908c0a@135.181.98.174:26656,74446e62d66daab41734bee243a9aee43cc9c009@65.108.199.187:26656,a7bab0f25d638f913f558fe9be16718b07cfccda@65.108.72.175:29655,86093241c8b102a6fe2f9a0836adab46bc078b6e@213.52.129.168:26656,c009938f56e54754a70bd6fd46b277c5c4854e56@65.108.11.180:26656,deffdfec77bee19f52c54b81b99b376d9814ce6f@80.240.24.175:26656,0ae4b755e3da85c7e3d35ce31c9338cb648bba61@164.92.187.133:26656,1aaa54c8afc9b1acd2421ad42ed6d29839ddd114@38.242.244.75:26656,9171167f490a333127af8bbe19879287d4721ce0@185.193.67.197:26656,06ebd0b308950d5b5a0e0d81096befe5ba07e0b3@193.31.118.143:25656,4330c56631adda7d64875859a019b3b3ddc6a5d7@16.163.74.176:26646,6df81ff87b61c665dc16035769ba83323a26c135@65.108.85.61:26656,87ea1a43e7eecdd54399551b767599921e170399@52.215.221.93:26656,19ade81d57a59b0279af31d57cd62e9a88a25109@195.3.221.131:26656,4a3a0e62750c959ff4e9569461d32a91d9fc5e04@65.108.233.175:26656,c57b8b5ce39260e75e38d048424c9417ac26c1cf@65.108.230.75:26656,5a8011bf946be1d2dd23647d3b515e08027b3eb6@5.189.149.36:26656,380fe8cd2764c4c0d04e8526d6c5addde5804d3d@130.61.122.244:26656,08d9025913badb9911d0a90b570240646e57e623@34.139.95.190:26656,e32ccdf863aaae227f35b4e552175c8f5a842cf4@159.223.218.77:26656,ad8f51565996dfd56aaef70d51a2efe85d72f83c@134.209.88.127:26656,7a7a0e149f7dff9d41664ce5e0b6d0098fccbdc1@65.108.75.237:26656,c3038d7fa344d94f79dfabc508a923893eab3a94@217.79.178.14:26656,7930d86f1863c4237a3e4c8d036a3033b08c8132@23.121.249.57:26656,d2127444daca86e343a336499fd6c32cd2ce243d@164.92.67.221:26656,df55e4d537403a0741fbe2ba95a5037c6608fe3f@128.199.54.28:26656,183e981a48801e84e6cd6cbb04d095e2c7a48393@107.21.89.71:26656,b690b0e6a904fc0172ef1eccc07bea9f48f4e454@141.94.73.39:53756,c80ab74906d37b5df87897ce8256f0895cfe5663@148.251.131.234:10256,7e126fcfba06b6901f509d312d56d4a72735ef31@23.175.144.185:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kujira/config/config.toml
```

## Update Min Gas Fees + Timeout Commit

```shell
# update gas prices

nano ~/.kujira/config/app.toml
minimum-gas-prices = "0.00125ukuji"

# update timeout commit

nano ~/.kujira/config/config.toml
timeout_commit="1500ms"
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
kujirad keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
kujirad keys add WhisperNode --recover

# Query the keystore for your public address
kujirad keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File ea3f8aca.md>)

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
