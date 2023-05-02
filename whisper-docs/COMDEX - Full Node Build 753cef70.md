# Install pre-requisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y
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
wget https://golang.org/dl/go1.18.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz
rm -rf go1.18.3.linux-amd64.tar.gz
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
git clone https://github.com/comdex-official/comdex.git
cd comdex
git fetch
git checkout v7.0.0
make install
```

To confirm that the installation has succeeded, you can run:

```shell
comdex version --long
# this will return commit 996939cd711d1c71276f347d27bbc25672dad401
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
export CHAIN_ID=comdex-1
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME="WhisperNode"
```

## Set minimum gas prices

In `$HOME/.comdex/config/app.toml`, set minimum gas prices:

```shell
minimum-gas-prices = "0.025ucmdx"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
comdex init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.comdex/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/comdex-official/networks/main/mainnet/comdex-1/genesis.json > ~/.comdex/config/genesis.json
```

Verify Hash - `88b5e9c2dc1b258791ea1fbbe0110b408f36fef7c62ffd2459b44cb000fb16d0`

```shell
jq -S -c -M ' ' ~/.comdex/config/genesis.json | shasum -a 256
```

This will replace the genesis file created using `comdex init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.comdex/config/config.toml`:

```shell
# persistent peers list
sed -i "s/persistent_peers =.*/persistent_peers = \"f74518ad134630da8d2405570f6a3639954c985f@65.0.173.217:26656,d478882a80674fa10a32da63cc20cae13e3a2a57@43.204.0.243:26656,61d743ea796ad1e1ff838c9e84adb38dfffd1d9d@15.235.9.222:26656,b8468f64788a17dbf34a891d9cd29d54b2b6485d@194.163.178.25:26656,d8b74791ee56f1b345d822f62bd9bc969668d8df@194.163.128.55:36656,81444353d70bab79742b8da447a9564583ed3d6a@164.68.105.248:26656,5b1ceb8110da4e90c38c794d574eb9418a7574d6@43.254.41.56:26656,98b4522a541a69007d87141184f146a8f04be5b9@40.112.90.170:26656,9a59b6dc59903d036dd476de26e8d2b9f1acf466@195.201.195.111:26656\"/g" "${HOME}"/.comdex/config/config.toml
```

## Set seeds list

Update our p2p seeds found in `~/.comdex/config/config.toml :`

```shell
# p2p seeds
sed -i "s/seeds =.*/seeds = \"aef35f45db2d9f5590baa088c27883ac3d5e0b33@3.108.102.92:26656,7ca14a1d156299999eba9c394ca060368022d52f@54.194.178.110:26656\"/g" "${HOME}"/.comdex/config/config.toml
```

---

# State Sync Node

*Bootstrap node from state-sync snapshot*

```shell
curl -s https://rpc.comdex.one/status | jq '.result .sync_info | {trust_height: .latest_block_height, trust_hash: .latest_block_hash} | values'
```

- Example output:

```shell
{
  "trust_height": "3549879",
  "trust_hash": "461420F85D8A7A9833B5A1C1E7FCC461AC10247B840C7DD3BB53AC687E3AC0BB"
}
```

*Set options in config.toml*

- Set seeds in `~/.comdex/config/config.toml`. Seeds node provide list of peers on which there is a snapshots:

```shell
seeds = "08ab4552a74dd7e211fc79432918d35818a67189@52.69.58.231:26656,449a0f1b7dafc142cf23a1f6166bbbf035edfb10@13.232.85.66:26656,5b27a6d4cf33909c0e5b217789e7455e261941d1@15.222.29.207:26656"
```

- Set trust\_height and trust\_hash values from rpc/status output:

```shell
[statesync]
enable = true
rpc_servers = "https://rpc.comdex.one,https://rpc.comdex.one"
trust_height = 622568
trust_hash = "49979F39F77EFC1A60829DC3C7610779A1D407DBD7748C0F0C36463554E7F6EA"
trust_period = "168h0m0s"
```

*Start comdex node*

- Start comdex by running below command. There won't be any data, but to be sure please run reset-unsafe as mentioned below.

```shell
comdex unsafe-reset-all
comdex start
```

## Checking Sync Status

After starting the junod daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block and you are safe to proceed to upgrade to a validator node.

---

## Create (or restore) a local key pair

Either create a new key pair, or restore an existing wallet for your validator:

```shell
# Create new keypair
comdex keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
comdex keys add <key-name> --recover

# Query the keystore for your public address
comdex keys show <key-name> -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

<br>

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
comdex tx staking create-validator \
  --amount 9000000ujuno \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(comdex tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ucmdx \
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
