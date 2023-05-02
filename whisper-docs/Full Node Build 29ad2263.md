---

Original guide can be found here:

[canine-mainnet-genesis/genesis-prep.txt at main ·...](https://github.com/JackalLabs/canine-mainnet-genesis/blob/main/instruc/genesis-prep.txt "canine-mainnet-genesis/genesis-prep.txt at main · JackalLabs/canine-mainnet-genesis")

Snapshot:

<br>

Block Explorer:

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
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
rm -rf go1.19.5.linux-amd64.tar.gz
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

# Build Daemon from source

```shell
git clone https://github.com/JackalLabs/canine-chain jackal
cd jackal
git checkout v1.1.2
make install
```

To confirm that the installation has succeeded, you can run:

```shell
canined version --long
```

# Configuration of Shell Variables

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

```shell
canined init YOUR_MONIKER --chain-id jackal-1
```

### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/jackal/genesis.json --inet4-only
mv genesis.json ~/.canine/config
```

### Configure Seed

Using a seed node to bootstrap is the best practice in our view. Alternatively, you can use [**addrbook**](https://polkachu.com/addrbooks/jackal) or [**persistent\_peers**](https://polkachu.com/live_peers/jackal).

```shell
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17556"/' ~/.canine/config/app.toml
```

## Set persistent peers + Seeds + Gas

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.canine/config/config.toml`:

```shell
PEERCOUNT=7
SEEDS=$(wget https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/seeds.txt -q -O -)
PEERS=`curl -sL https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/peers.txt | sort -R | head -n $PEERCOUNT | awk '{print $1}' | paste -s -d, -`
GAS="0.002ujkl"

sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.canine/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"$GAS\"/" $HOME/.canine/config/app.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
canined keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
canined keys add WhisperNode --recover

# Query the keystore for your public address
canined keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to setup cosmovisor and start the node.

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
canined tx staking create-validator \
  --amount 1000000ujkl \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(canined tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --identity=""
  --fees 0ujkl \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
canined tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.canine/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
