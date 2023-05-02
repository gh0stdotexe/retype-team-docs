---

Original Guide:

[Sourced Installation and Setup - Source Protocol](https://docs.sourceprotocol.io/nodes-and-validators/sourced-installation-and-setup "Sourced Installation and Setup - Source Protocol")

Block Explorer:

<https://explorers.cryptech.com.ua/sourcechain-testnet>

Polkachu Testnet Support: Source Protocol

[Source Testnet | Polkachu](https://polkachu.com/testnets/source "Source Testnet | Polkachu")

Github Repo:

[GitHub - Source-Protocol-Cosmos/testnets](https://github.com/Source-Protocol-Cosmos/testnets "GitHub - Source-Protocol-Cosmos/testnets")

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
wget https://golang.org/dl/go1.18.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.18.5.linux-amd64.tar.gz
rm -rf go1.18.5.linux-amd64.tar.gz
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

Install Ignite CLI

```shell
curl https://get.ignite.com/cli! | bash
```

### Write permission[​](https://docs.ignite.com/guide/install#write-permission "Direct link to heading")

Ignite CLI installation requires write permission to the `/usr/local/bin/` directory. If the installation fails because you do not have write permission to `/usr/local/bin/`, run the following command:

```shell
curl https://get.ignite.com/cli | bash
```

Then run this command to move the `ignite` executable to `/usr/local/bin/`:

```shell
sudo mv ignite /usr/local/bin/
```

On some machines, a permissions error occurs:

```shell
mv: rename ./ignite to /usr/local/bin/ignite: Permission denied
## may display
Error: mv failed
```

In this case, use sudo before `curl` and before `bash`:

```shell
sudo curl https://get.ignite.com/cli! | sudo bash
```

# Build Daemon from source

```shell
git clone -b testnet https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
ignite chain build
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
sourced version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=sourcechain-testnet
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
sourced init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.source/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcechain-testnet/genesis.json > ~/.source/config/genesis.json

# verify sha256sum
sha256sum ~/.source/config/genesis.json
# 2bf556b50a2094f252e0aac75c8018a9d6c0a77ba64ce39811945087f6a5165d
```

This will replace the genesis file created using `sourced init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.source/config/config.toml`:

```shell
PEERS=ff52cead20cf60f6cc02757152b681af50869d0d@116.203.191.53:26656,3874444d8504dc31836c1e41ffe408bee6b80904@38.242.159.243:39656,ebba276ee4bb406cc31be3c1ca15dd8384dda779@178.18.243.159:26656,c675b39ca0e47a95c4d37c8a98a14ac93b46bad6@143.110.184.196:26656,9243201cc50ad3c68bac3177388981b40e47a5af@204.12.248.74:26656,bd2545bdf9e111633140ad4b093d434af99d63a3@38.242.241.69:26656,07b6fa3e91b7bed1c47a46648089e8041641f050@178.18.240.148:26656,a7ce1512d39d9c9b1293e8b55b59d18f5313b536@116.202.236.115:20056,a1994aac20a3ab40b3cb2092e4e9f50397985856@185.196.20.37:26656,7545bbb24348add3b0f373e62d30ea796637ceec@95.216.223.200:26656,8f77417e770602ecd84c4e46c03b3153771a4add@115.73.213.74:27656,3620fb4df9be7e697f451f6331eb58eefff04e95@176.124.205.215:26656,be875169367c1b74fd2fddf12f1047a3661de1f5@173.249.38.51:26656,f0242fa41d044952aaa9acf9266aeff2cf7d5569@172.105.17.117:26656,c11b85deb59574812a7e6b9d6181df36bef15d2f@65.108.105.48:27656,6721349b8a9846b730fdadb9ddf0b0596acf98e7@217.13.223.167:26656,05dbcd1bb0563107c5eeb98a8da9d6cd9197bfcd@65.21.129.95:21756,ba0a2fac080e40fb33eeb59710d17003ef4d0de6@209.145.48.32:26656,66aca0db6580ae259a510f65bc221ca6601341ad@113.161.82.243:27656,9d16b552697cdce3c8b4f23de53708533d99bc59@165.232.144.133:26656,7ccb6331c8f8aed3439fe2f88d01f8808564f9e2@164.92.98.14:26656,7143126daf3c0983745a0b10b83c8e794c4fb2fc@65.108.126.46:33656,ba8620d21a266cefc7c14fbc70cb070efabb9920@65.21.227.78:26656,1bc2fa179a4df7f51b6fa27288454d0452e23442@62.171.144.222:39656,da549dbcd8507c52b492f3a64401a518d324725f@164.92.98.12:26656,fc1a626fce5f5eb93ceaad9ce08aed7d3b921e93@86.48.1.141:39656,e59efee95ba6e57f57918cfbbc5268b248b229e2@34.222.210.0:26656,38140b9335b8891b6bec8c49198490cf80287c78@95.216.218.186:26656,f268f469cd7fc88abcf5115a007ded1c2c912f1f@144.76.224.246:36656,14b16a1393ff0c95ce0a563e2225fc6e414edda7@159.65.94.178:39656,c749b47c438842d9874b515de130dfb11431360f@147.182.211.27:26656,b20497b3fb86603d04e00024766ec07dc3fe7e48@65.108.76.44:11564,e1a0a3e32f89e33c7f8f775dd2db704288aba39b@13.36.178.25:39656,fae907ab505bfd41fc2499bd002fd58adc6fc68a@173.249.26.69:26656,4ec8efac171577e8f6f04980a70c86f5261602f6@62.171.150.241:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.source/config/config.toml
```

## Update Min Gas Fees

```shell
# update gas prices

nano ~/.source/config/app.toml
minimum-gas-prices = "0.025usource"
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
sourced keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
sourced keys add WhisperNode --recover

# Query the keystore for your public address
sourced keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 6e44bd2d.md>)

---

## [Polkachu State-Sync Server Setup](https://polkachu.com/testnets/source/state_sync "Source Testnet Node State-Sync | Polkachu")

Our state-sync RPC server for Comdex is:

```shell
https://source-testnet-rpc.polkachu.com:443
```

## Instruction

We assume that you use Cosmovisor to manage your node. If you do not use Cosmovisor, you will need to customize the following instruction slightly.

Create a reusable shell script such as **`state_sync.sh`** with the following code. The code will fetch important state-sync information (such as block height and trust hash) from our server and update your **`config.toml`** file accordingly.

```shell
nano source_state_sync.sh
```

```shell
#!/bin/bash

SNAP_RPC="https://source-testnet-rpc.polkachu.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.source/config/config.toml
```

Grant user the privilege to execute the script and then run the script:

```shell
chmod 700 source_state_sync.sh
./source_state_sync.sh
```

Stop the node:

```shell
sudo service cosmovisor stop
```

Reset the node:

```shell
# On some tendermint chains
sourced unsafe-reset-all

# On other tendermint chains
sourced tendermint unsafe-reset-all --home $HOME/.source
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
sourced tx staking create-validator \
--amount 1000000000usource \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details "validators write bios too" \
--pubkey=$(sourced tendermint show-validator) \
--moniker “<key-name>” \
--chain-id sourcechain-testnet \
--gas-prices 0.025usource \
--from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, keybase.io id, etc. To see a full list:

```shell
sourced tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.source/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
