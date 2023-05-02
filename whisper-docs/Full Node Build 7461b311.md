---Original Guide:[](https://docs.google.com/document/d/1M0heml037f0SxmTV1ARQMwb6sdBJorwTk_wR2FjOxxI/edit#)Block Explorer:[Ping Dashboard](https://testnet-explorer.brocha.in/sei%20atlantic%202/uptime "Ping Dashboard")Polkachu Testnet Support: Sei<br>Brochain - Atlantic-2 Testnet Support:[Sei Atlantic 2 testnet services | Brochain](https://brocha.in/testnet/sei-testnet-2/services/ "Sei Atlantic 2 testnet services | Brochain")<br><br>---# Install pre-requisites```shell
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
seid version --long | grep commit

# commit hash
bb91480231f2c3112931110445ceb22015148c87
```# Configuration of Shell VariablesFor this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.## Choose the required mainnet chain-idChoose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:```shell
export CHAIN_ID=atlantic-2
```Then source it:```shell
source ~/.zshrc
```## Set your moniker nameChoose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:```shell
MONIKER_NAME=<moniker-name>

# Example
MONIKER_NAME="WhisperNode-0"
```# Setting up the NodeThese instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.## Initialize the chain```shell
seid init $MONIKER_NAME --chain-id $CHAIN_ID
```This will generate the following files in `~/.sei/config/`- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`## Change Node Mode```shell
sed -i -e 's/mode = "full"/mode = "validator"/' $HOME/.sei/config/config.toml
```## Download the genesis file```shell
curl https://raw.githubusercontent.com/sei-protocol/testnet/main/atlantic-2/genesis.json > ~/.sei/config/genesis.json
```This will replace the genesis file created using `seid init` command with the mainnet `genesis.json`.## Set persistent peersUsing the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.sei/config/config.toml`:```shell
PEERS=65c257f9275beb1b99ca169ef89743c034b15db0@3.76.192.224:26656,650a118a5919c1d0eb3d9f17b14cfb2a6b1c8b9d@3.120.150.255:26656,5710d992d9c33b01c3b23df4cbd715e9b4c7b46b@3.71.0.14:26656,272699de6f61eb4a7509ae33fa49af0d4bf13784@18.192.115.237:26656,862b03573172a3366afe1cabb903ba0552689e63@198.244.228.59:11956,1a71212a12e67dbf0d1b557e54251676f8f8af6b@65.21.200.7:7000,762cf4f35aec09857df14ac2bc78824f8f89d5db@65.109.28.219:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.sei/config/config.toml
```## Create (or restore) a local key pairEither create a new key pair or restore an existing wallet for your validator:```shell
# Create new keypair
seid keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
seid keys add WhisperNode --recover

# Query the keystore for your public address
seid keys show WhisperNode -a
```**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**---# Setup Cosmovisor / Service FileFollow the instructions to set up cosmovisor and start the node.[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File f2118423.md>)---## Download Testnet Snapshot (Atlantic-1)[Sei testnet snapshots | Brochain](https://brocha.in/testnet/sei/snapshot "Sei testnet snapshots | Brochain")### Stop the service and reset the data```shell
sudo systemctl stop seid
rm -rf $HOME/.sei/data
```### Download the latest snapshot```shell
SNAPSHOT_FILE=$(curl -Ls https://snapshots.brocha.in/sei/atlantic-1.json | jq -r .file)
curl -L https://snapshots.brocha.in/sei/$SNAPSHOT_FILE | lz4 -dc - | tar -xf - -C $HOME/.sei
```### Restart the service and check the log```shell
sudo systemctl restart seid
sudo journalctl -u seid -f --no-hostname -o cat
```---# Upgrade to a validator> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*To upgrade the node to a validator, you will need to submit a `create-validator` transaction:```shell
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
