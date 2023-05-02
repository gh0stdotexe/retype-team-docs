---

Original Guide:

[Joining Incentivized Testnet - Sei](https://docs.seinetwork.io/nodes-and-validators/seinami-incentivized-testnet/joining-incentivized-testnet "Joining Incentivized Testnet - Sei")

Block Explorer:

[Sei explorer by Nodes.Guru](https://sei.explorers.guru/validators "Sei explorer by Nodes.Guru")

Polkachu Testnet Support: Sei

[Sei Testnet | Polkachu](https://polkachu.com/testnets/sei "Sei Testnet | Polkachu")

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
mkdir -p ~/quasar-backup && cd ~/quasar-backup
wget https://github.com/quasar-finance/binary-release/blob/main/v0.0.2-alpha-11/quasarnoded-linux-amd64?raw=true
cp 'quasarnoded-linux-amd64?raw=true' ~/go/bin/quasarnoded
sudo chmod +x ~/go/bin/quasarnoded
```

The `<version-tag>` will need to be set to either a [testnet `chain-id`](/validators/joining-the-testnets#current-testnets) or the latest [mainnet version tag](/validators/joining-mainnet).

To confirm that the installation has succeeded, you can run:

```shell
quasarnoded version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

## Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join. Set the `CHAIN_ID`:

```shell
export CHAIN_ID=qsr-questnet-04
```

Then source it:

```shell
source ~/.zshrc
```

## Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```shell
MONIKER_NAME=<moniker-name>

# Example
MONIKER_NAME="WhisperNode-0"
```

# Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network, and upgrade your node to a validator.

## Initialize the chain

```shell
quasarnoded init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.sei/config/`

- `genesis.json`
- `node_key.json`
- `priv_validator_key.json`

## Download the genesis file

```shell
curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json > ~/.sei/config/genesis.json
```

This will replace the genesis file created using `seid init` command with the mainnet `genesis.json`.

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers`  + `seeds` in `~/.sei/config/config.toml`:

```shell
PEERS=f64bb73bfe46de6a351a3778bfdbac97f4545b1f@135.181.251.99:26656,f55359b8aab9f4bc34460b56651d557db5ac3542@135.181.251.102:26656,dc174ea8fd846d708ca01831002633c07bd07deb@95.217.211.174:26656,3de20b80b0230642cb3415a18ba22642e5456523@65.108.138.183:8095,d07e26c0c638f2d7caa535fa182edbb28835b2ec@15.235.11.129:27212,849348eec39535f9c5688354ad5f4cd7f234ccc1@65.108.221.215:26656,830953e60981b9d77a4bd2d17820ecd11872abfc@65.108.201.154:3030,55471ab816f651ca80a3a09d58b4833ac3661b7a@38.242.156.245:12656,7f037abdf485d02b95e50e9ba481166ddd6d6cae@185.144.99.65:26656,15c967fcf7f4b631e6725e8eaa23c3de31063351@135.181.138.161:26656,c391d717d5e7d4b7410893dea3bf71385561fd65@31.187.74.204:26956,fc2e2a09a34b378e9f24c63204bfbe6fb9c038b0@135.181.251.100:26656,312ec945f98f2839e14b41b4b3e8e1a53aa78537@135.181.147.1:26656,81273358060a44e8db686082d704578375f79609@65.21.207.188:26656,e50ed55db8e3b49678ffc7221e3ba94634c9b767@138.201.141.76:46656,e5bdbc9b8f8b3a8b7530f88dcfd1b2c73d6c0ded@88.99.56.219:36656,0825cc4ee1c577b8ad513487ff0603155d166eac@194.163.146.31:12656,2b11ed8b37907542c813e4d85371319dfa3d0928@159.69.125.201:26656,f8820c97bd6c3d86a25984690c2f7304f39d6eb4@162.55.175.5:26656,303b1a27fedd717fa911dbd5920359fafa0dc331@95.217.158.236:26656,9da519dd5a8f6d49a1449aec0179e2f5a4946366@165.227.140.24:26656,c0084c2c6a21a4a520f79db940ddd7243afafdbf@135.125.180.36:20156,2ba10c09dbc4f4f3a6e47d47d1e39d0f81c29743@195.68.159.62:26656,74e18c0ecf9e7c7e20b8079d5434bc0d6ad39c75@162.55.223.216:26656,91fbb4bcf5e862cf65fcf0aae0dea43ad9474d0c@141.94.139.233:26656,1b1dccf59d4ceb4c66e6db968263334a230f8f0a@162.55.223.23:26656,6c198d7c26283d0f35e8ccc832744ea4fa77b93b@213.239.218.199:26631,4e64eead0e0e896a963d564a581efce0d833fbe3@65.21.132.27:29656,e7d8a6cda6b1b2c4bc6293dd7cea6ebdf6736622@178.63.74.39:26656,a0f8913e9128032ceae5c63d9fee69f3a77aa9b6@159.69.126.18:26656,a9717ba4b4533af494677a7dbd27298be8c96f49@109.107.191.48:12656,aaa1da62895d2a8daaf09b235ca82a55c8d9efd7@173.212.203.238:46656,98370c36129f6d7f85f46e5ed24859595e3bdf01@194.146.12.50:10143,eff641714985f8966393f9c1e3350b20d15e3c70@38.242.146.6:26656,f6d497e5aa89c975a93852800c9a4232eb12c275@168.119.157.196:12656,843e6c0e496646733c07d9ccaec8b0ff2a090af9@45.77.67.113:26656,62ecc461e078d4bd74b1e6aa0497a4ed5cdcb8c7@46.63.100.22:12656,7748bd92e8f30e772208f539a8b696611b5686f5@75.119.135.34:26656,27effcd26238cf987b2cf6695c1d961a59d833ba@38.242.198.5:12656,18f9f8aa7a6a32807635e9cdebb5d8cb939b0cbf@95.214.52.206:26656,17c09a1033799859ef4e17f87e2cb2baa3368001@5.161.139.197:26656,1ea5ec28d6cf3cb7c9b3fca3dac42d772077ba9a@5.161.98.82:26656,635c32b8c21b35b62570b4155fa9425f8e4358cf@5.161.105.13:26656,c22205240014c51d9c5126c3840315cda50275fd@54.241.77.154:26656,94290dd1f8944ed3450f007637d0099d5bb248b9@45.126.125.173:26656,49b1ba399bd43f98fc43b7024fc855568af46d68@54.180.190.199:26656,a598bb0a5ee7381210a61722f9c2f3fee456a16e@54.183.197.142:26656,c79f189cfd98b60a49db416622d0e7c1417639c4@65.21.205.38:26656,b1f7e49b8fd8565cab4cb4c4a0d365c5aeb19c38@65.21.225.178:26656,5ea0fd2f544e0ef3e54956e98bf720bb170f2cef@54.151.43.184:26656,294144594209ab5b8e20add5644c7078e3f58f83@135.181.211.97:26680,7c5d26f019f1430ecbe3f69bb62aea176ca399f9@65.21.7.111:26656,1204900bc58bd491cb8077159b498a14ea9cb019@194.163.146.32:12656,3e51e41ee801a75025deaa899e404b23944be55c@45.85.147.140:12656,90916e0b118f2c00e90a40a0180b275261b547f2@65.108.72.121:26656,59d61b09374c7b6c5c7c4eab3caba14c9158d0eb@85.10.244.226:26656,77363bf144dc152700daf3cd80a38e053457eb51@65.108.130.189:26656,2e1db8e6b318655780381a7d6459f2699cb9fa43@15.235.53.182:27212,d263b81b76bfcb9f8ccc1657a303b59c3f102976@135.181.201.209:26656,3590457f1396f2a9a26537eddcaea50bec69342b@65.108.193.210:7095,b1c2c950331fd2f7d076744a3997e0dc2c586f93@51.195.189.48:20036,c886b3e988a5f2e054c71991c5afb48a3eb5a80c@49.12.234.38:26656,25de2c94130c66232d65e17e2529f70608ce824b@5.161.109.155:26656,20aa22d0409980cbdc0e07c15f04e8d234f403f6@95.216.70.104:26656,94b6fa7ae5554c22e81a81e4a0928c48e41801d8@88.99.3.158:10956,d47df41ed5dff54bf152c3585937a58413d6addd@38.242.140.131:26656,8536f475269173ef632affe42f1a558c78fdd168@116.203.229.214:26656,b52eb2396ca46a9bc560551bfa020bbd7c65cc60@144.76.168.83:26656,bbcfb3d66e0b9875d260c0a5f825f98bab9a4ac4@173.212.222.167:14656,dd846c7cab125a1af531d834e99abcf8d3d42c92@65.108.52.192:26656,688726ee964b0c1a102e1a8ca169a5e2089fc4a7@65.108.45.200:26858,e78551829dc38df18971f00b0f2054d67237d8fc@207.180.226.203:26656,345ee8f81489d6c1b47a1fc14d43c76044826d21@65.109.17.29:26656,55f6dab1d295d3dca01db72198c4904da76448cf@45.8.144.44:26656,f6dfccf50f508a4b2b1f83e04ee7781471392e97@5.161.96.255:26656,1981aeec73ef0bbf59cddfce9cf4eddce43b3c53@217.76.159.104:26656,7f0960fb4cb46877036cd9bbb7b5b4c0d428a25b@65.108.204.119:46656,6beca0870eccca09d0e35152d5916a87573cdd0c@65.21.248.220:12656,34e082537b50ef6d7363ac827f3a6e487c5d1bca@95.216.1.249:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.sei/config/config.toml
```

## Create (or restore) a local key pair

Either create a new key pair or restore an existing wallet for your validator:

```shell
# Create new keypair
seid keys add WhisperNode

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
seid keys add WhisperNode --recover

# Query the keystore for your public address
seid keys show WhisperNode -a
```

**After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.**

---

# Setup Cosmovisor / Service File

Follow the instructions to set up cosmovisor and start the node.

[Setup Cosmovisor / Service File](<Setup Cosmovisor _ Service File fa5a9659.md>)

---

## Download Testnet Snapshot (Atlantic-1)

[Sei testnet snapshots | Brochain](https://brocha.in/testnet/sei/snapshot "Sei testnet snapshots | Brochain")

### Stop the service and reset the data

```shell
sudo systemctl stop seid
rm -rf $HOME/.sei/data
```

### Download the latest snapshot

```shell
SNAPSHOT_FILE=$(curl -Ls https://snapshots.brocha.in/sei/atlantic-1.json | jq -r .file)
curl -L https://snapshots.brocha.in/sei/$SNAPSHOT_FILE | lz4 -dc - | tar -xf - -C $HOME/.sei
```

### Restart the service and check the log

```shell
sudo systemctl restart seid
sudo journalctl -u seid -f --no-hostname -o cat
```

---

# Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```shell
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
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
seid tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it is damaged or lost in some way. Please make a secure backup of the following files located in `~/.sei/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
