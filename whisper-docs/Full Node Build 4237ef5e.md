---Original Guide (deprecated):[Joining Testnets - Sei](https://docs.seinetwork.io/nodes-and-validators/joining-testnets#adding-a-validator-node "Joining Testnets - Sei")Block Explorer:[Sei explorer by Nodes.Guru](https://devnet.sei.explorers.guru/validators "Sei explorer by Nodes.Guru")Team Google Doc (Instructions):[](https://docs.google.com/document/d/1fa7EqAt3EbfrtL_nW6gNvkKnNJ0h1DW_Vp8RTkQBx90/edit#)Snapshot Downloads:[Sei Devnet testnet snapshots | Brochain](https://brocha.in/testnet/sei-devnet/snapshot "Sei Devnet testnet snapshots | Brochain")---# Install pre-requisites```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y

# install toolchain and ensure accurate time synchronization
sudo apt-get install make build-essential gcc git jq chrony -y
```# Install GoFollow the instructions [here](https://golang.org/doc/install) to install Go.For an Ubuntu LTS, we can use:```shell
# find location of existing GO (if any)
which go
go version

# remove old GO if existing
sudo rm -rf /usr/local/go

# install updated GO
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm -rf go1.20.linux-amd64.tar.gz
```Unless you want to configure in a non-standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.```shell
nano ~/.zshrc
```Add the "export Pathing" rules  at the bottom, and then save the file:```shell
# add export PATHS below
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin:$GOBIN
```After updating your `~/.zshrc` you will need to source it:```shell
source ~/.zshrc
```# Build Daemon from source```shell
git clone https://github.com/sei-protocol/sei-chain.git sei
cd sei
git checkout 2.0.40beta
make install
```The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).To confirm that the installation has succeeded, you can run:```shell
seid version --long

# should output 2.0.40beta
```## Configure Node```shell
# generate a 32-bit hex address for moniker
openssl rand -hex 16

# copy output of above command and add to MONIKER=""
MONIKER=""

ln -s $HOME/.sei/cosmovisor/upgrades/2.0.40beta $HOME/.sei/cosmovisor/current
sudo ln -s $HOME/.sei/cosmovisor/current/bin/seid /usr/local/bin/seid
seid config chain-id sei-devnet-3
seid init $MONIKER --chain-id sei-devnet-3
curl -Ls https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-devnet-3/genesis.json > $HOME/.sei/config/genesis.json
sed -i -e "s|^bootstrap-peers *=.*|bootstrap-peers = \"f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@sei-devnet.seed.brocha.in:30515,babc3f3f7804933265ec9c40ad94f4da8e9e0017@testnet-seed.rhinostake.com:51956\"|" $HOME/.sei/config/config.toml

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001usei\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning *=.*|pruning = \"custom\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-keep-recent *=.*|pruning-keep-recent = \"3000\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-keep-every *=.*|pruning-keep-every = \"0\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-interval *=.*|pruning-interval = \"10\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^snapshot-interval *=.*|snapshot-interval = \"1000\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^snapshot-keep-recent *=.*|snapshot-keep-recent = \"2\"|" $HOME/.sei/config/app.toml
```> **IMPORTANT: **
>
> This will generate the following files in `~/.sei/config/` - **BACK THESE UP**!- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`---## Create (or restore) a local key pairEither create a new key pair or restore an existing wallet for your validator:```shell
# Create new keypair
seid keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
seid keys add WhisperNode --recover

# Query the keystore for your public address
seid keys show WhisperNode -a
```**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**---# Setup Cosmovisor / Service FileFollow the instructions to set up cosmovisor and start the node.[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File 10b674cb.md>)---# Upgrade to a validator> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*To upgrade the node to a validator, you will need to submit a `create-validator` transaction:```shell
seid tx staking create-validator \
  --amount 1000000usei \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(seid tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025usei \
  --from WhisperNode
```The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, or keybase.io id, etc. To see a full list:```shell
seid tx staking create-validator --help
```---# Backup critical filesThere are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.sei/config/`:- `priv_validator_key.json`
- `node_key.json`It is recommended that you encrypt the backup of these files.
