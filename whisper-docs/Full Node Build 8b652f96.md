---

### The original guide can be found on Github:

[genesis-tools/LAUNCH.md at main · terra-money/genesis-tools](https://github.com/terra-money/genesis-tools/blob/main/LAUNCH.md "genesis-tools/LAUNCH.md at main · terra-money/genesis-tools")

### Snapshots:

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
# checkout and install
git clone https://github.com/terra-money/core
cd core
git checkout v2.1.4
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
terrad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=phoenix-1
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

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
terrad init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.terra/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget https://phoenix-genesis.s3.us-west-1.amazonaws.com/genesis.json
mv ./genesis.json ~/.terra/config/genesis.json
```

This will replace the genesis file created using `terrad init` command with the mainnet `genesis.json`.

## Set persistent peers + Gas

Using `sed`, we can set the `persistent_peers` + `minimum-gas-prices` easily:

```shell
sed -i'' 's/minimum-gas-prices = ""/minimum-gas-prices = "0.15uluna"/' $HOME/.terra/config/app.toml
sed -i'' 's/persistent_peers = ""/persistent_peers = "dc865a0d882f30e41e99ef23d9e6164163607523@54.147.79.192:26656,bdce6030a2bdebe4c660a76599fe3dee4a42d50f@35.154.54.64:26656,0f1096278efafcf3f0d3bd5b6544e6b8dcc36a0e@206.189.129.195:26656,c8ab8910e5f7bfcc6e81351eb851eb8c0540a194@exitnode.cereslabs.io:26656,33afc1c21cb225bb2cfb9700442a576bbaeb7691@163.172.100.203:26656,9038d63588e0ab421fa71582720c1efb1ee867f6@45.34.1.114:27656,daa2fd0dc725d6673e7688c9c57fc3b6d99c83c4@solarsys.noip.me:27656,331c2bbcd1aab921563dce85dedae840e1369e39@142.132.199.98:10656,91b675be5f81931375358e02ab687c88fab02e41@135.148.55.229:11656,9dc9e9b50c4cae52cdbec2034d879427b2a429ae@54.180.81.122:26656,ad825ef6b29306d80b0eb8101133cedf7933eb5e@116.203.36.94:26656,f2069012aec5ced4e88e7e4311391eabe72bb5a3@node-phoenix.terra.lunastations.online:26656,9fe9cc32880be11134e1ec360a61082541019233@162.55.85.23:26656,065fd6f49a4a424727433b3a8e3d5945e4935d9c@78.46.68.41:26656,9909a0fc254852005c6d382b2321008e669f7ad0@65.108.199.18:9095,5d423d17f84f25b8c2167fd8be4abd0b8ba89091@65.108.110.125:12095,1b308820fa76c033cd2e41e1a11b2533a55f4a03@65.108.10.95:17095,1903ccc818ee9923cd66078689d71000b2c6e4c9@65.108.98.218:10095,276dd792b8eb8703e65edcb5f5527d16b762191c@138.201.16.240:15095,7d7e614db9a73e1e6d7b56903ad79bfc13623783@142.132.252.8:11095,f58748aa96bd0110a1ed5c00b0fbf4e9faf92b20@18.219.130.4:26656,1e5e39efe018876c355cbb81a772668e5ce5e1ea@52.54.234.245:26656,49171079e358230f214f521b572f4b0caa4afa1b@51.222.42.166:26322,9c888a18a8c4a493830f56e4effd5da6035acfb7@141.95.66.199:26322,4ebf87085c2a3cc65d09549938985cf72a3c7734@phoenix.terra.kkvalidator.com:26656,89e8c6097c1cc56e2f1e0f96ef2eedd3dd31aa8c@34.67.180.86:26656,69e8b793059b813d79e5c3f4c4ff9f0d0c7d82e8@216.153.60.190:26656,f1818c94e9fdf77027196f96fe9062002d588c2a@216.153.60.189:26656,cdf150244e95ad655d36b12ddf1a7884f5e3072a@34.64.196.228:26656"/' $HOME/.terra/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
terrad keys add WhisperNode

# Restore existing terra wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
terrad keys add WhisperNode --recover

# Query the keystore for your public address
terrad keys show WhisperNode -a
```

==After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place.The seed phrase is the only way to restore your keys.==

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to set up cosmovisor and start the node.

---

# Syncing the node

After starting the daemon, the chain will begin to sync to the network. The tcopyime to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network's current block and you are safe to proceed to upgrade to a validator node.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
terrad tx staking create-validator \
  --amount 1000000uluna \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(terrad tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id phoenix-1 \
  --gas-prices 0.025uluna \
  --from WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
terrad tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.terra/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
