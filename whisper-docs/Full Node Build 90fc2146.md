---

Block Explorer:

[Interchain Explorer by Cosmostation](https://www.mintscan.io/stride/validators "Interchain Explorer by Cosmostation")

Github Repo:

[GitHub - Stride-Labs/stride: Stride: Multichain Liquid Staking](https://github.com/Stride-Labs/stride "GitHub - Stride-Labs/stride: Stride: Multichain Liquid Staking")

Polkachu Support: Stride

[Stride Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/stride "Stride Validator Node Snapshot | Polkachu")

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
git clone https://github.com/Stride-Labs/stride.git stride
cd stride
git checkout v5.0.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
strided version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=stride-1
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
strided init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.stride/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/stride/genesis.json --inet4-only
mv genesis.json ~/.stride/config
```

This will replace the genesis file created using `strided init` command with the mainnet `genesis.json`.

## Persistent Peers:

Here is a script for you to update **`persistent_peers`** setting with these peers in **`config.toml`**.

```shell
PEERS=cbbc8c1c9da23b71ccd2138fbf41bb710afe44b1@65.21.170.3:28656,9731222819ddacf2b0238e51527aa95156a04b06@57.128.133.22:26656,c427e6b2b9ae3c58f3578c5bad398a469ed3c903@65.108.142.47:36656,cfd27429d382ecf366ddad02c88f15a8753092c8@66.172.36.135:28656,0e67ce079f4e26ad5d22d7b1ba61e7df214d626c@65.108.101.50:60556,c4cb7a93147a4ec0eb3ed7bd6a6c3d9ce68fa2f8@78.47.222.208:26639,67e90a4956250adf62a2e257f569d80907d11d86@141.95.103.115:26656,ffe6f90207382788eb43aebb11e2581455f57b90@167.235.138.94:26656,f02b8862ab5a9add71148340cc28d765fba8069a@138.201.220.51:26666,07565037957f0eab414f85b16f36db458d8b34dc@5.9.13.162:20086,27e3200f2b3f83c403ad9dfa09bf83ae73b179b3@149.102.143.220:10173,23180f90318d0003a4e8140a1e67407bf874d69d@78.107.234.44:25656,7e0a230dfbecb877f05fe9f1dde6c91d3b633c43@95.216.142.94:26656,04ea9eceee16db90872fee3fbef9ac50a87702c5@185.248.24.29:26656,74b693b1b0745d250becfbdb550d36504e03bf92@93.115.25.15:26656,ec4f27b82691f44d38b4801889b3b92643bdc5c2@185.144.99.234:26656,cb0b38aa612e8ac05f704d9b2feb7526607afb77@159.203.191.62:26656,a63421772de24812d62a842ffb40cd93946ab1df@194.163.136.90:26656,edb5e5a80654041e2a6e83852b75f8325e88bfd2@34.71.129.146:26656,59a13b0e8ce91c6d507b06c09b0ed44a1574cad3@54.177.215.240:26656,f5732d5a406bdbbf08acad017c0993c0aa8ebe70@35.247.108.210:26656,f400b2b2665c62d3ba64a940a4c08a7db874ff6a@34.94.25.244:26656,b549e0f88cbebe6cfd3f772937a70640b950fd98@66.172.36.133:28656,2f02a4012f90f5d1a9a85748dd9aa14155ed4a71@66.172.36.134:28656,d3cee85a10d72b6f4eb40f544323acd104b29c23@51.68.44.219:23856,b5f9fa874781f975687018ae559f0d952d3a2e24@52.52.208.179:26656,e821acdaf0c7a3c60ea3cd4eb4a98a62dad06f58@43.201.12.41:26656,e0b4c670b35cc0cf16b570ea3397a5f785c9d1b6@65.109.48.57:26656,99237ee23683d1bf4cb426a7042793464d8f5401@128.199.15.159:31437,f93ce5616f45d6c20d061302519a5c2420e3475d@135.125.5.31:54356,9063fce4a1fc50185b2cafd56bc8176a45072c09@57.128.133.23:26656,7ef5ff00fe94933b8ba4b7ae4a8632ece5db11df@104.198.4.170:26656,9ee75491e354965d8bfd8434aa093f8613bc1dce@65.108.238.103:12256,609251c15cd2330f13436a23cb185d7f50aac707@34.171.123.54:26656,9c69539f7d6db4f5637c64a1552677fcd0c83c7d@34.170.162.50:26656,ea30fb0d90563efd98131d7778e7a53d0e9ad633@116.202.236.115:21016,4d44c8d6cd2dc2fae0b607c380b040b3651e81d6@95.214.53.27:12256,1387946c04bceb472113f657f55f670f71709230@65.108.4.188:12256,84ff28824a911409e2c24f2f5ede87ae1b859b5f@5.9.147.22:26156,6a1087004245692128a6ad11b812bb3640955b86@162.55.235.69:25656,97e90b4d658dc7b10c05e387b1384f6670f64943@83.136.249.85:26656,4b177aeb56846ef7b56b0078c09e6b326276880e@34.135.162.111:26656,1f5518caf7fdefab0a9f9fab848843231e414af8@3.69.24.186:26656,076e97f47762a477f2ae3dd3e798a7970b6bb20d@52.52.110.228:26656,65ae054d22c83eb586fe4b175b52564d5383a80f@158.160.11.206:26656,3f96c5efb65249b62551043c6b3328280c15ea6a@34.130.75.249:26656,c4688bb34164eacacaa374bc7440b87986dd87ac@162.251.235.252:26656,a757fc9ea95a7f643d392ec9fdaa31cbf06e76d9@195.3.221.21:12256,98004cf40a82aa7015d5f4fba75564201e62de26@141.95.124.151:21016,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@65.108.233.109:12256,cb91a11588d66cfd9c01f99541df4978a08e0e39@34.65.7.252:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
```

## Addrbook for Stride

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/stride/addrbook.json --inet4-only
mv addrbook.json ~/.stride/config
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
strided keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
strided keys add WhisperNode --recover

# Query the keystore for your public address
strided keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File e0c31a03.md>)

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:etup Cosmovisor / Service File￼

```shell
strided tx staking create-validator \
  --amount 1000000ustrd \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(strided tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas="auto" \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, keybase.io id, etc. To see a full list:

```shell
strided tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.stride/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
