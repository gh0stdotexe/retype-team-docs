---

GentX Guide:

[mainnet-artifacts/genesis-validators-guide.md at main ·...](https://github.com/aura-nw/mainnet-artifacts/blob/main/xstaxy-1/genesis-validators-guide.md "mainnet-artifacts/genesis-validators-guide.md at main · aura-nw/mainnet-artifacts")

Running a Fullnode:

[Docs](https://docs.aura.network/validator/running-a-fullnode/ "Docs")

Seeds, Peers, etc:

[Aura - kjnodes | Chain services](https://services.kjnodes.com/home/mainnet/aura "Aura - kjnodes | Chain services")

Block Explorer:

[TC Network | We Are The Cosmos Validator](https://explorer.tcnetwork.io/aura/validator/auravaloper1dyw4kvnfzj9geh75jlw4z2tgmj5g9f35wu63pp "TC Network | We Are The Cosmos Validator")

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

Unless you want to configure in a non standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder.

```shell
nano ~/.zshrc
```

Add the "export Pathing" rules at the bottom, and then save the file:

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
git clone --branch aura_v0.4.4 https://github.com/aura-nw/aura
cd aura
make
```

To confirm that the installation has succeeded, you can run:

```shell
aurad version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

We can generate a 32-bit moniker using the following command:

```shell
openssl rand -hex 16
```

```shell
aurad init 3AC395EF184023C5DDA497EF0B0BC8EF --chain-id xstaxy-1
```

## Prepare configuration to join Mainnet

You should now update the default configuration with the Mainnet's genesis file and application config file, as well as configure your persistent peers with a seed node.

```shell
curl -s https://raw.githubusercontent.com/aura-nw/mainnet-artifacts/main/xstaxy-1/genesis.json > ~/.aura/config/genesis.json
```

You can also run verify the checksum of the genesis checksum:

```shell
jq -S -c -M '' ~/.aura/config/genesis.json | sha256sum
# this should return
# 90b9404d38167e3b40f56ddc11a1565f0107b89008742425e44905871699febc  -
```

### Persistent Peers

We can add these `persistent_peers` to our `config.toml`:

```shell
sed -i "s/persistent_peers =.*/persistent_peers = \"d9bfa29e0cf9c4ce0cc9c26d98e5d97228f93b0b@65.109.88.38:17656,a58b4dec687b60ba05cf9a3e4cd1181b09c0661f@65.109.93.152:34656,edbd221ceecf4e0234fb60d617a025c6b0e56bf0@178.250.154.15:36656,3e7ef25f1c9829351936884618659167400eb0f1@142.132.149.171:26656,0179528068da0dfaf61005cf5aa28793ca42b129@85.25.74.163:26656,e46238ddcf2113b70f59b417994c375e2d67e265@71.236.119.108:40656,a859027129ee2524b57c43b9ecbe3bcc4d120efb@142.132.195.58:26656,65bf908c6c41cacfce9652ed69a17337b023d0d0@57.128.85.172:26656,ed68064620cebd196f56335bf801144efa9fb5ef@185.22.232.82:26656,f43c7c9a194ee5a97665a9aad8f887fdbb75e4ca@65.109.225.86:46656,358b375d2ed068e5670301760476637aa9ad79a0@51.79.19.15:30656,a19b89ebbf7331f435b8ef100ce501d2377922ea@209.126.116.182:26656,5d9146e9446df65ac30dd0a2dcb7e5887aaa6fa6@188.40.67.160:26656,a1f949c765bfc493ddd2e0e8477170bcc3b86a57@194.163.179.176:16656,f67f9a6f5121b6388c84812a812d5d6eca0b39e8@148.251.66.248:26656,7ff603bf2eb8249b9a1e695a232d99fdaf8a0f13@195.201.197.159:26156,abb367c73ef28fc90f5071e1258a23c0e5be17cd@103.107.183.89:26656,dc9c2ab4055a2ef8ddca435e9d8c120969562f98@194.247.13.139:26656,ed15ae05f17dd4e672eec0a96c38364d063b68dc@65.108.6.45:60756,eec4c706ee03921d103018647a4695706bc91b21@13.212.73.184:26656"\"/g" "${HOME}"/.aura/config/config.toml"
```

### Set up minimum gas price

```shell
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uaura\"|" $HOME_PATH/config/app.toml
```

## Addrbook for Aura

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.kjnodes.com/aura/addrbook.json
mv addrbook.json ~/.aura/config
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
aurad keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
aurad keys add WhisperNode --recover

# Query the keystore for your public address
aurad keys show WhisperNode -a
```

> **After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](https://app.nuclino.com/t/b/cc174b4c-761a-4836-9f73-48ce977317d1) instructions to setup cosmovisor and start the node.

---

# Upgrade to a validator

> **Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.**

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
aurad tx staking create-validator \
  --amount 1000000uaura \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(aurad tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uaura \
  --from <key-name>
```

> The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
aurad tx staking create-validator --help
```

<br>

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.aura/config/`:

- `priv_validator_key.json`
- `node_key.json`

> It is recommended that you encrypt the backup of these files.

<br>
