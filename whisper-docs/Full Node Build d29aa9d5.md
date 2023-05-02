---

Original Guide:

[Run a Node - Kujira Docs](https://docs.kujira.app/run-a-node "Run a Node - Kujira Docs")

Block Explorer:

[Kujira explorer by Nodes.Guru](https://kujira.explorers.guru/validators "Kujira explorer by Nodes.Guru")

Polkachu Snapshots:

[Kujira Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/kujira "Kujira Validator Node Snapshot | Polkachu")

Polkachu Addrbook:

[Kujira Addrbook | Polkachu](https://polkachu.com/addrbooks/kujira "Kujira Addrbook | Polkachu")

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

# Build Daemon from source

```shell
git clone https://github.com/Team-Kujira/core kujira
cd kujira
git checkout v0.7.1
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
export CHAIN_ID=kaiyo-1
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
# WAITING FOR FINAL GENESIS FILE
cd ~/.kujira/config
curl -s https://raw.githubusercontent.com/Team-Kujira/networks/master/mainnet/kaiyo-1.json > ~/.kujira/config/genesis.json
```

This will replace the genesis file created using `kujirad init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.kujira/config/config.toml`:

```shell
PEERS=9813378d0dceb86e57018bfdfbade9d863f6f3c8@3.38.73.119:26656,ccffabe81f2de8a81e171f93fe1209392bf9993f@65.108.234.59:26656,7878121e8fa201c836c8c0a95b6a9c7ac6e5b101@141.95.151.171:26656,0743497e30049ac8d59fee5b2ab3a49c3824b95c@198.244.200.196:26656,338d79e24ce36a9580ce3e9fce8eeb84e0e6f17b@[2a01:4f9:6b:2e5b::3]:26656,2efead362f0fc7b7fce0a64d05b56c5b28d5c2b4@164.92.209.72:36346,d24ee4b38c1ead082a7bcf8006617b640d3f5ab9@91.196.166.13:26656,5d0f0bc1c2d60f1d273165c5c8cefc3965c3d3c9@65.108.233.175:26656,5a70fdcf1f51bb38920f655597ce5fc90b8b88b8@136.244.29.116:41656,35af92154fdb2ac19f3f010c26cca9e5c175d054@65.108.238.61:27656,e65c2e27ea06b795a25f3ce813ed2062371705b8@213.239.212.121:13656,f6d0d3ac0c748a343368705c37cf51140a95929b@146.59.81.204:36656,ecafd5cadaf3526a588550a7bc343ce2670c988d@185.16.39.231:26656,afc247bceddc0eeeb6cf62db6fb4f985b03dd3b0@65.108.74.21:26656,94b124a422113f1871c3ea750097842004e4a095@18.222.185.33:26656,3c6e0c7b8be14ccf1717d84f3c11dcc1d2bfcba9@65.108.232.149:30095,f709e13a7fbbd34afe000e5534bf024fc981a16b@65.108.76.242:31095,00d610b86d500740bd41c9db0955865680138fdb@5.9.98.56:32095,2c0be5d48f1eb2ff7bd3e2a0b5b483835064b85a@95.216.7.241:41001,76b2b62f39d121c8248e6f61543c13830fd756af@77.68.22.110:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kujira/config/config.toml
```

## Update Min Gas Fees + Timeout Commit

```shell
# update gas prices

nano ~/.kujira/config/app.toml
minimum-gas-prices = "0.00119ukuji,0.00150factory/kujira1qk00h5atutpsv900x202pxx42npjr9thg58dnqpa72f2p7m2luase444a7/uusk,0.00150ibc/295548A78785A1007F232DE286149A6FF512F180AF5657780FC89C009E2C348F,0.000125ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.00126ibc/47BD209179859CDE4A2806763D7189B6E6FE13A17880FE2B42DE1E6C1E329E23,0.00652ibc/3607EB5B5E64DD1C0E12E07F077FF470D5BC4706AFCBC98FE1BA960E5AE4CE07,617283951ibc/F3AA7EF362EC5E791FE78A0F4CCC69FEE1F9A7485EB1A8CAB3F6601C00522F10,0.000288ibc/EFF323CC632EC4F747C61BCE238A758EFDB7699C3226565F7C20DA06509D59A5,0.000125ibc/DA59C009A0B3B95E0549E6BF7B075C8239285989FF457A8EDDBB56F10B2A6986,0.00137ibc/A358D7F19237777AF6D8AD0E0F53268F8B18AE8A53ED318095C14D6D7F3B2DB5,0.0488ibc/4F393C3FCA4190C0A6756CE7F6D897D5D1BE57D6CCB80D0BC87393566A7B6602,78492936ibc/004EBF085BBED1029326D56BE8A2E67C08CECE670A94AC1947DF413EF5130EB2,964351ibc/1B38805B1C75352B28169284F96DF56BDEBD9E8FAC005BDCC8CF0378C82AA8E7"

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

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File b1fa9e0a.md>)

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

SNAP_RPC="https://kujira-rpc.polkachu.com:443"

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

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.kujira/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
