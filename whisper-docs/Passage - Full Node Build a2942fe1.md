---

*Original guide can be found on the Stargaze Github:*

[mainnet/passage-1 at main · envadiv/mainnet](https://github.com/envadiv/mainnet/tree/main/passage-1#node-requirements "mainnet/passage-1 at main · envadiv/mainnet")

*Snapshot links:*

N/A (yet)

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

Unless you want to configure in a non-standard way, then set these in the `.zshrc` in the user's home (i.e. `~/`) folder

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
git clone https://github.com/envadiv/Passage3D passage
cd passage
git checkout v1.1.0
make install
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](https://www.meisternote.com/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](https://www.meisternote.com/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
passage version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=passage-1
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

## Set minimum gas prices

In `$HOME/.passage/config/app.toml`, set minimum gas prices:

```shell
nano ~/.passage/config/app.toml

# update gas prices
minimum-gas-prices = "0upasg"
```

<br>

---

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.

## Initialize the chain

```shell
passage init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.passage/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/passage/genesis.json --inet4-only
mv genesis.json ~/.passage/config
```

This will replace the genesis file created using `passage init` command with the mainnet `genesis.json`.

Confirm Checksum:

```shell
jq -S -c -M '' ~/.passage/config/genesis.json | shasum -a 256

# should output
b7816525b8856d0fdd7e1410124b933d254facd22d0a2f8210d5eedea76b0ede
```

## Set seeds list

Update our p2p seeds found in `~/.passage/config/config.toml :`

```shell
# p2p seeds
seeds="aebb8431609cb126a977592446f5de252d8b7fa1@104.236.201.138:26656,b6beabfb9309330944f44a1686742c2751748b83@5.161.47.163:26656,7a9a36630523f54c1a0d56fc01e0e153fd11a53d@167.235.24.145:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.passage/config/config.toml
```

Set Persistent Peers

Update p2p peers found in `~/.passage/config/config.toml`:

```shell
PEERS=8e0b0d4f80d0d2853f853fbd6a76390113f07d72@65.108.127.249:26656,0111da7144fd2e8ce0dfe17906ef6fd760325aca@142.132.213.231:26656,56e129d2bc23bc58639173542f9c449818c24c45@65.109.31.114:2450,ba9fe6f1df3707c0e5dbfa84bba4c2754b98e437@65.108.126.34:26656,c3287cf29f940a08bb5129c26c403e09eebaf695@65.109.34.14:46656,6a37ad91a09d62e26e9cdafd718471897c5d8062@135.181.219.23:26656,9e6f10f401dadcb28a8164574147056b8e3ab748@65.109.34.42:26656,f6d876d71392ca36a3edf9f10540e31b37788c45@65.109.38.208:26656,a98ef597e3154bacd490a27efdf6eaad7b31046c@65.108.140.109:56156,b2d425eb33e0bd3b1aa35e75526072e1ed117215@95.217.35.111:27656,fc41c96371d0ae091d300bf1eb26ee3ea6a3d465@217.79.252.58:26665,36bb54bfef16ce46669fa54bb2013d694a99deda@168.119.89.31:56156,c2be0927ff17ef8b9b9d8c9313a32f1e8fac9048@142.132.194.134:26656,b0c006a6e3e38146d6c84703a597f8e7a73f032f@135.181.132.248:26656,f76ccaa550d283ef1adb55c6aca5d94eab6a1806@146.59.81.204:24456,0d9ef5067933cfcca67ad70173c2ca6e4dd89d89@146.19.24.23:26656,98f8513e38098049e59b068a12f8ba21b3e1347b@46.101.241.248:26656,facdfc6431106b2caa926ce39ed201a88b486763@38.146.3.138:26656,813f91ac5338de44e28aa7946039986dbec286ff@198.244.228.17:36056,621f75a74a95298fe16e0c2dd899c087bcba6594@65.108.195.29:50656,f9469c6be21f4fc66ab8cb823483649af355b84a@51.79.177.229:26661,8a210f1bcfc9015a7bc18dcc5add29c0dce3f2dc@135.181.173.59:26656,471518432477e31ea348af246c0b54095d41352c@88.198.129.75:26656,3ba83db0090669d6bcd639b6e849b748bed35629@95.179.209.69:26656,960f6f00956ebc227096fc0edcdbe1b0ff1e76f1@192.53.117.12:26656,b6beabfb9309330944f44a1686742c2751748b83@5.161.47.163:26656,b9bf6ccfa526a8d6c87ee3a3087954f0f4c8fea5@65.108.238.183:26631,3b57543e38b69ab1c175e62f5b65df24dd11202f@65.21.230.230:46656,afd2d71bc80218d59a53863d72221af31fe8da71@65.108.238.103:11656,57e44db08eeaa1acfb324aa44fcbd5b049a20232@65.109.39.61:26656,5057428876c966fc99984d2a5a0f677b8e847f9c@136.243.40.41:26656,f11d76f7c50a904b725a905d0685d15bb9724398@136.243.95.80:26656,f627b4204f9c38a46a27304b1c0981643c98351e@142.132.147.56:26656,c124ce0b508e8b9ed1c5b6957f362225659b5343@136.243.208.27:26656,6aa09bd3aa20acbb46e9d000f52e305046201086@139.144.77.106:26656,3dd55b4b89308ad7e617a8d68be1b28d861e6b16@65.108.77.106:26829,0d23f5c54c55188bf1cb3081b624e98a7e76708d@15.235.46.17:26656,be86a665d809469fdf0c5adac17c940ddffe166e@75.119.157.167:31656,f66de874ea9fe9a74da9d0383ec1ef81f2a0c0a0@50.18.254.83:26656,6b0e3f9dca79e3a13046fef2e040492aa5c736c3@65.108.99.37:26686,50ee8b6194e48de810befc250eb1138e06413562@13.52.121.13:26656,7a9a36630523f54c1a0d56fc01e0e153fd11a53d@167.235.24.145:26656,8e4e1f1e087c76c71c64e477e95495833da82aa2@135.181.172.187:26656,b3710004185e388635fbc5dfb4901ed7494337ac@23.88.66.239:36156,eb1ac31dd0500e8e9a6ca5d2a79bf93397d6b131@167.235.28.48:26640,9723e0234226acda5649c06e97a4f04e6f66bec8@144.76.63.67:26776,3279c84b10f06a64182df1a53e5461bb6a0df58e@135.181.115.111:28656,424aad6476fbad8073ea28ce9d80296f3f45286c@128.199.224.193:26656,a1316ab986d5a752566922d27367a0a3c97a0265@194.163.130.2:26916,2b95c6213daf063dd50850f261395923ccded63e@159.223.64.126:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.passage/config/config.toml
```

## Addrbook for Passage

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can decide to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/passage/addrbook.json --inet4-only
mv addrbook.json ~/.passage/config
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
passage keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
passage keys add WhisperNode --recover

# Query the keystore for your public address
passage keys show WhisperNode -a
```

> **After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

<br>

---

# Setup cosmovisor

Follow the [2. Setup Cosmovisor](https://app.nuclino.com/t/b/cc174b4c-761a-4836-9f73-48ce977317d1) instructions to setup cosmovisor and start the node.

---

# Upgrade to a validator

> **Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.**

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
passage tx staking create-validator \
  --amount=9000000upasg \
  --pubkey=$(passage tendermint show-validator) \
  --moniker="<your_moniker>" \
  --chain-id=passage-1 \
  --commission-rate="0.05" \
  --commission-max-rate="0.10" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --from=<your_wallet_name>
```

> The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
passage tx staking create-validator --help
```

<br>

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.passage/config/`:

- `priv_validator_key.json`
- `node_key.json`

> It is recommended that you encrypt the backup of these files.

<br>
