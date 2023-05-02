---

Polkachu Install Guide:

[Juno Node Installation Guide | Polkachu](https://polkachu.com/installation/juno "Juno Node Installation Guide | Polkachu")

Polkachu Snapshot:

[Juno Validator Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/juno "Juno Validator Node Snapshot | Polkachu")

Polkachu State Sync:

[Juno Node State-Sync | Polkachu](https://polkachu.com/state_sync/juno "Juno Node State-Sync | Polkachu")

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
git clone https://github.com/CosmosContracts/juno
cd juno
git fetch
git checkout v12.0.0
make install
```

To confirm that the installation has succeeded, you can run:

```shell
junod version --long
# this will return commit b75e783651c860273b40c4c9a0ba438f8c9989d0
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=juno-1
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

Use the random hex generator found here to generate a 32-bit hash for our moniker:

<https://numbergenerator.org/hex-code-generator#!numbers=1&length=32&addfilters=>

```shell
MONIKER_NAME="0FD76EDFD1364A29F22F61111B012E90"
```

## Set persistent peers

Persistent peers will be required to tell your node where to connect to other nodes and join the network. To retrieve the peers for the chosen mainnet:

```shell
# Set the base repo URL for mainnet & retrieve peers
CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/$CHAIN_ID" && export PEERS="$(curl -s "$CHAIN_REPO/persistent_peers.txt")"
```

NB: If you are unsure about this, you can ask in discord for the current peers and explicitly set them in `~/.juno/config/config.toml` instead.

## Set minimum gas prices

In `$HOME/.juno/config/app.toml`, set minimum gas prices:

```shell
nano ~/.juno/config/app.toml

# update gas prices
minimum-gas-prices = "0.0025ujuno"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
junod init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.juno/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/juno/genesis.json --inet4-only
mv genesis.json ~/.juno/config
```

This will replace the genesis file created using `junod init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.juno/config/config.toml`:

```shell
PEERS=c2bbd82cd749261a37e9863fac0c02a8906bdf44@65.108.232.150:33095,c44a49da8adf6ab86a26c9b7fa53da179597605b@85.10.243.90:26656,45ce9b4a653d1ffcba833d0c07eabaeee86b6a92@162.55.131.238:15610,b2c932d42a81c529328fe968eee387d7918faae2@162.55.232.44:26656,97f76286b6815e84c6bd457beb7c59d821c0b852@141.95.72.198:26656,d08fcf530e50d76b803aa2372c4bfaee1a032508@195.201.193.224:26656,77a62d703a87a783e18599bc7fea781de02e7967@5.9.97.174:15610,f3cee9895a0be20067b1aa2ca3fd7ede59ee0b71@83.149.102.56:33095,bfeae5219b310065f4785fddaa96aca69e66b0e6@141.98.219.163:26656,e7c642bdd79fd79cd2677f4f8b1351236b5ec2f3@204.16.241.208:26656,f90ab44ae1c38915a99d0c1605b64989d543ad72@51.81.208.145:26656,1e95f780f110ca2335ecd09dca1927a9b5bb0090@154.12.241.136:26656,8b4c6c339b9df66f7189b976cb86fd7b30d92f33@136.243.93.134:33095,777aaec971cd7e0bc7d8ea37c734d9075aa092e8@195.154.107.51:26656,0ed395467f8a74dfaa8c72d8e34234bfc3f36746@49.12.176.140:26656,c7789b5dab5ea2d1112dda74200e1385e9df8353@34.71.221.174:26656,5f25d965e5dfab10d71ecc8948cebbc6112443ad@42.3.204.146:2221,f0eebd04aa8236a55e5d373dfe1a6406742310fc@51.91.215.170:33095,0d9949d16394ea8423bf637bbc67d67b23b14058@122.155.165.127:26656,29b7ac8939ad9272f24849b259d7d2764e252151@52.213.191.239:26656,78a56aab17bbd2155a1367cfa7bb7096e5f1c6ed@65.109.28.75:26656,c5a2f8d48055b4b9f2f3e231900d2b0e48159d34@95.216.17.104:27655,093bb41c023236cc1d2e9218067fc3fd59362fc8@65.109.67.119:26656,a787b5091f4740cde316384087b5f1deecbcceaf@148.251.89.143:28050,dd27ebc420b68841450b40f3c41b754c44472732@188.34.144.13:26656,b49e28c215096323f20dba05929f9f4c1f6b58a8@65.108.142.81:26664,28aac28c5de2fb26b0ba23221e9173d9cd1a614c@65.108.11.167:29519,0f6f2e50d57d30a10eea3eb3efe49986157ce7ed@185.187.170.18:26656,75da2dbd926508f82ba74b6763155227ff3c8070@65.109.33.108:26656,3aa00177a41ff8cedebf9f3f7f4358c1795ff3ea@185.207.250.206:50016,d83892be2e6efc38e255943ce86ae8229d2aee90@178.128.220.188:26656,bb9d8fc631450587dfb90baead7286d59f258db2@35.215.18.128:26656,9135a94392c4793e89902554e9446df708e14a18@176.9.7.37:26656,134d8c28fdfac3f38fd9db7ee8f7685b3564a413@142.132.248.214:26656,4f4e7c2a4081b8c7c12ee0aae5736cacfd951976@138.201.158.63:26620,4a0991be53ff3d42f0cd9e4c7fc3eed3d7584ce8@213.133.100.164:26656,e16814ffd2d184f726f67ab587205a02e7596dac@195.201.63.87:34756,45f4da091b7f7536c3e0182083ff2326d0c3be6a@66.85.137.122:26656,650d04a352266084f2f37bd9f89eb0a9b892049a@198.244.213.94:22356,a543dcf56f6c70562667e0debe48310789f151dc@78.138.46.104:26656,60902900200add0071fb4efd23e1d72f20f8ef21@65.108.98.235:28856,4deec88245f034335552e0f615d4f76c52c31948@93.189.30.111:26656,d7587af3ec9d29f0c3742aebeb17d2463b07f4c5@5.161.125.124:36656,7f593757c0cde8972ce929381d8ac8e446837811@178.18.255.244:26656,2832bdb0a1bdddb2b17d1229a799290222c085d0@135.125.189.131:33095,08c31507bcf5140c559435bcbb2619e4c3675be1@195.3.221.21:12656,37eeafbe61d4a0b32421d20578df7eb06ebeeba2@168.119.50.205:36656,d398f4668a564f3b245c489e44b26145ad513edf@162.55.253.38:26656,9f8cd938d81d4232517ac1d29bd1510e3aac5ce4@146.59.52.95:33095,46af91c713ab4119b1f938528877299edb631a7d@5.161.49.37:36656,aab8ab60cdf7b12e88661d65cd1ef7d78905951b@66.172.36.139:11656,336286d0f3dd1f4e3116b226e101acbba3b7b51e@149.202.72.170:26620,531601a86419128f5d2752c13da8d378d3e84a4e@95.217.121.45:26656,0099e13cdaf9f9c88aadc2fad6271dc10a2860c4@88.208.242.3:26656,f999c72243565d6805cb864025d9d3f3bdc199c5@65.108.108.179:27756,44294f4bbeeb751d6a2f1d84dfeb8b9224f5acb8@135.181.116.165:26656,f790b6c1ed813471b9ef78f3f6b77ac7f12a1fb2@144.126.217.120:26656,ea9c1ac0e91639b2c7957d9604655e2263abe4e1@185.181.103.136:26656,6a565453d1d3dc69d911c7ac0c8b568c8ed33173@35.187.188.128:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.juno/config/config.toml
```

## Addrbook for Juno

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/juno/addrbook.json --inet4-only
mv addrbook.json ~/.juno/config
```

## Create (or restore) a local key pair

Either create a new key pair, or restore an existing wallet for your validator:

```shell
# Create new keypair
junod keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
junod keys add WhisperNode --recover

# Query the keystore for your public address
junod keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

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
junod tx staking create-validator \
  --amount 9000000ujuno \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(junod tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ujuno \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
junod tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.juno/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
