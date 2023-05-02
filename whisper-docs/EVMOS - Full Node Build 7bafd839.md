Testnet Guide:

[Join a Testnet | Evmos Documentation](https://docs.evmos.org/validators/testnet.html "Join a Testnet | Evmos Documentation")

Testnet Explorer:

[Interchain Explorer by Cosmostation](https://testnet.mintscan.io/evmos-testnet "Interchain Explorer by Cosmostation")

GitHub Repo (Testnet):

[testnets/evmos\_9000-4 at main · evmos/testnets](https://github.com/evmos/testnets/tree/main/evmos_9000-4 "testnets/evmos_9000-4 at main · evmos/testnets")

Testnet Faucet:

<https://faucet.evmos.dev/>

EVM Explorer (Testnet):

[Evmos Testnet evmos\_9000-4 Explorer](https://evm.evmos.dev/ "Evmos Testnet evmos_9000-4 Explorer")

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
wget https://golang.org/dl/go1.19.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
rm -rf go1.19.2.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

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

After updating your `~/.profile` you will need to source it:

```shell
source ~/.zshrc
```

# Build Daemon from source

```shell
git clone https://github.com/tharsis/evmos.git
cd evmos
git checkout v6.0.1
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
evmosd version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](/validators/joining-mainnet#mainnet-chain-id). Set the `CHAIN_ID`:

```shell
evmosd config chain-id evmos_9000-4
```

## Optional: Set minimum gas prices

In `$HOME/.evmosd/config/app.toml`, set minimum gas prices:

```shell
minimum-gas-prices = "0.025aevmos"
```

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
evmosd init WhisperNode-0 --chain-id evmos_9000-4
```

This will generate the following files in `~/.evmosd/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

For Evmos genesis, we need to grab the new file and replace our existing with it:

```shell
cd ~/.evmosd/config
rm -rf genesis.json
wget https://archive.evmos.dev/evmos_9000-4/genesis.json
```

This will replace the genesis file created using `evmosd init` command with the mainnet `genesis.json`.

Next, we'll validate the `evmosd` genesis file:

```shell
evmosd validate-genesis
```

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.evmosd/config/config.toml`:

```shell
# persistent peers list
PEERS=1601d5ca76ffa87cbf97e251dd63f21b90f003f5@65.108.71.92:32656,b6d79086b1df2850b283df4de8d39dc1685de2d7@65.108.79.246:26664,c454065d8a35cb4652474cb67cf6430e38ae6824@65.108.125.55:11456,c2859af47c7c43a09e10f93c0f0258d4987a8970@94.130.23.149:26656,3a6b22e1569d9f85e9e97d1d204a1c457d860926@35.233.98.138:26656,94a1c3990a213cc18c2847bc186a599b29568935@74.118.140.25:26656,fdc27e32b386b9cdb438cc7559c2966389aebb8f@34.116.232.56:26656,6dd65bbc54f3a0f7c1001e85d405e57b44b9f355@5.161.93.45:26656,9151cf912e0d550238b19ca1c16d900c9a497e54@3.239.123.131:26656,b1aa949df47ba05310f72fb8b7a5430816dd2dd4@52.87.207.127:26656,e8efea0d6bebe8d55240707aef3cccd6394e0f86@161.97.74.88:26651,42be1c21eed6dc2b38153912d17652b4e8fddd06@194.163.165.176:26656,f5a83f20b60ea6b8ccf7031e617e6587165d60ca@54.174.3.241:26656,134841b65113eaf271b370475fc177626dc1f026@3.87.41.100:26656,4e37e23c50fe52f7c57c61874c2469db0c77dfff@65.21.230.183:26656,9922b7206c885bae3ae2076586b111f36795c373@162.55.90.254:29656,649234798458b493deece6a9e8ba1dc42b2c9f0f@65.108.238.61:22656,f4c1ae5aee205b6c704480da85733cad25056ae0@176.9.167.181:26656,33098ee0e6a5dab0eae6e5704be12037743792e5@51.75.62.98:26656,0a2dba495fdb5d292a21be1840499f1d29612602@195.201.105.242:26656,3fc932c471f6cf6c33e1d4790193bcbb4b92cbff@34.132.40.252:26656,cf609ae142ad5edf8b1e53d32410f3063e2cc323@104.196.213.235:26656,3a394d316a501d4a14d69077bc0e87d92db77a97@161.97.87.10:26656,19455bb0e9dff278f4e4a4b3c6205a601888c907@65.108.98.124:60206,3fc5fc7a850a4160995a25f9026e673cb363c903@164.92.130.241:26656,e0660cdd60f919fa69ace31f22715698c85f7366@80.64.208.116:26656,a933529c47f50997c7b5473e3cefc2bab811d69a@144.217.74.11:27656,44283f2bd3cd6d5a994187174c73a8ea3bc46756@161.97.151.40:26656,a9113c2fff8170391bfdf8f6a4e395339486f7b2@108.209.99.67:26621,2f820112ef204e61e1b081851b42c6312dc17f7c@65.108.67.148:26636,366bb843ad850e1a72f0f49cf347321c30d2a4de@108.209.102.187:26642,c5e0e571771fb97bc93cca3824402f069eb6f838@135.181.37.63:26656,8f100fbcc2dec641de48ce5ffdae121b1eff3551@3.23.63.191:26656,3f006c3d02fbc3784e941868644d045b70432817@34.139.161.222:26656,c00ec5f0c8d664e5eccdb312a7394f16b325e725@34.85.200.127:26656,63595a0bb9e94f52d25cfcbd0fafc3b5c4af8282@65.108.137.124:26656,8d1a0934d38f09e587cca498c9f552abf3258806@34.76.223.126:26656,eeebc671f1a174739dd0a4d9777b7cab5d1be8e0@65.21.237.194:26656,a5ba1f7bafee5706328a01da33286920fa981afd@194.163.176.91:26656,4ae4f1c4ac51b234872381a9f40ff7ff480409fe@51.159.36.3:26656,5216de0eae13e0904bcacf3c879eb27bab6c6826@3.14.188.240:26656,f848f63d139abe26ff452c780e3cd0df6b699421@65.21.126.182:26656,75b1597bae31217b66ee4b1cb0f284724c02639b@188.34.188.67:26656,7246f2b23f63696e9d2acb234fb3d7140bc07e3a@65.108.239.181:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
```

## Set seeds list

Update our p2p seeds found in `~/.evmosd/config/config.toml :`

```shell
# p2p seeds
SEEDS=`curl -sL https://raw.githubusercontent.com/tharsis/testnets/main/evmos_9000-4/seeds.txt | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" ~/.evmosd/config/config.toml
```

---

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
evmosd keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
evmosd keys add <key-name> --recover

# Query the keystore for your public address
evmosd keys show <key-name> -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor

Follow the [2. Setup Cosmovisor](<2. Setup Cosmovisor cc174b4c.md>) instructions to set up Cosmovisor and start the node.

<br>

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
evmosd tx staking create-validator
  --amount=1000000000000aevmos
  --pubkey=$(evmosd tendermint show-validator)
  --moniker="WhisperNode🤐"
  --chain-id=evmos_9001-1
  --identity="9C7571030BEF5157"
  --website="https://WhisperNode.com"
  --details="WhisperNode operates robust, high up-time validators across multiple Cosmos-based blockchains. We are highly active participants in the communities we validate for as we believe this is a necessity for effective on-chain governance and staying up to date on all network issues."
  --commission-rate="0.05"
  --commission-max-rate=echo $DAEMON_NAME_8"0.20"
  --commission-max-change-rate="0.01"
  --min-self-delegation="1000000"
  --gas="auto"
  --gas-prices="0.025aevmos"
  --from=WhisperNode
```

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
evmosd tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.evmosd/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
