---

GitHub:

[NodeStake Explorer](https://explorer.nodestake.top/canto/staking "NodeStake Explorer")

Original guide can be found here:

<https://docs.canto.io/technical-reference/validators/quickstart-guide>

Snapshot:

[Canto Node Snapshot | Polkachu](https://polkachu.com/tendermint_snapshots/canto "Canto Node Snapshot | Polkachu")

Block Explorer:

[NodeStake Explorer](https://explorer.nodestake.top/canto/staking "NodeStake Explorer")

---

# Install prerequisites

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

Unless you want to configure in a non-standard way, then set these in the `.zshrc` in the user's home (i.e., `~/`) folder.

```shell
nano ~/.zshrc
```

Add the “export Pathing” rules  at the bottom, and then save the file:

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
git clone https://github.com/Canto-Network/Canto.git canto
cd canto
git checkout v5.0.0
make install
```

To confirm that the installation has succeeded, you can run:

```shell
cantod version --long
```

# Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell `.profile`, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME`. Shell variables should be named with ALL CAPS.

## Configure Node

### Initialize Node

Please replace **`YOUR_MONIKER`** with your own moniker.

```shell
cantod init YOUR_MONIKER --chain-id canto_7700-1
```

### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```shell
wget -O genesis.json https://snapshots.polkachu.com/genesis/canto/genesis.json --inet4-only
mv genesis.json ~/.cantod/config
```

## Set minimum gas prices

In `$HOME/.cantod/config/app.toml`, set minimum gas prices:

```shell
sed -i.bak 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0001acanto"/' $HOME/.cantod/config/app.toml
```

## Set persistent peers

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.cantod/config/config.toml`:

```shell
PEERS=744294d2ecf5ddf14065be6d325e68dcbdf0c646@66.172.36.136:51656,f9fc759eb2fa4eb2159825cae149ba1065efa236@66.172.36.134:51656,81f89cfa6dd6ec4cb2ee297e67dd4613657c4194@88.198.32.17:30656,bea21c6cc721726a486dbd7f14c5e81ee12f6eaa@35.83.23.119:26656,f9f8f88dfde1bacca2f152089bb20c600dbb9d04@43.204.152.200:26656,905b48ff27d52e002f12252ad67e5a0a979a5bfb@35.188.27.40:26656,2d7826e04685c4afb7baf6a045a3098c1306e1cc@5.9.108.156:35095,8cb9419ede1d830e78b4dd1318bdbd4e6be000d7@144.76.27.79:36656,f886849e7f563c5c3e4f5a666be76a2184b246ef@85.10.203.235:26676,c78ddd65c25aadd6e99f998dad36f22e3b418ab8@65.21.170.3:32656,a441b9fec8006f28fb2add0517fa823b886834d6@5.79.79.80:35095,1d3ab5cc05452e29d8dafb4f96fcf3841c485287@51.210.223.185:35095,9723b0dac535d9e5c28e62413ddda54386ff8955@138.201.249.155:26656,78e7fb7977290f81a51ae636ed06ae1952d4263e@65.108.141.109:16656,8b2ac4899b5a0b6e289850bde707f45421d1e9a4@213.239.207.175:30656,43393ba9763a9b1b95785330c5059811e5ed7f91@95.217.122.80:17656,685c48cbc2ba54e20f49645d48b0878d6944d8e4@65.109.94.221:32656,978a3730fc791492c009ff380d8e8bb25997da1b@65.109.65.210:29656,b8cc93a20982f6e7dd0201757c642d2ddc76eee9@148.251.53.202:26656,484e252942ffcc0c6e31278ac0f47a3ca1317aef@142.132.238.165:26656,174f015f606fd1f139447158b81a1824f6352854@65.108.75.107:16656,ab88f189db7825f376050a034d8bf0028442cfc3@34.89.161.101:26656,61c8c3dce43e7221a5dab1a3c86366f34d2edddd@213.239.215.77:26656,876a17aa48201ec9b8937d81e28b44bfcb4d318c@15.235.115.149:10004,885c91e45319265422f42b63940fc7f01330569b@23.117.63.185:26656,f724d16c43147bad59a036f243aa79c6f4455d2d@23.88.69.167:26858,cdad27c5be53788cbf42dd1336adeabc253b6e52@38.242.251.238:26656,82956d94714ded8fd785acb498a0aeb7aafad7ff@85.17.6.142:26656,5401995b201605a03d9e1fd0460cbef49218bbf5@65.108.126.46:32656,6ed040a6d393738c1bbeebd200c2e2f660614907@135.181.222.179:28656,bedcf918f53967cd37a0d03e67997d1b40c6c152@5.161.113.61:26656,8f21e61a2a81c96e5b761b256cd5c9d13a325281@86.48.2.82:26656,f74639c33b7647b0462e634974147c20505747a6@213.239.216.252:23656,9188d4b9b9e1a7e86ac6a0e6bee343e4c5f6fa25@114.32.170.200:26656,e6d62aa5215719eb1b7434e19bca4e7f62923ef4@65.108.106.172:58656,edb42b3caf26aa9c37362bb53f3d0e6038683ead@34.134.123.237:26656,c9e39b78c37b1bd360676d1e68f40a1f6c36d528@109.236.86.96:36656,69c21a89c74d08cb4a3c463dc813fe279fe4f080@51.79.160.214:26656,4fb5a871a1f263752da75e323e2ed73ed315a17f@95.214.52.138:26666,5e55b3bbab81818dd7c9e0c34c25f64377e2ea6c@104.248.2.163:26656,54316791649b65af344432bf4bd31f46df0cb79c@51.195.234.49:27756,eda9ae1a8a598db165f7db9932267343385f5873@209.145.56.78:36656,64172382922c636354387436d7e3b494b1abf577@46.10.221.196:26656,5ce67581ef51b30c70212a870f2e5ede27c31929@65.109.20.109:26656,ebd18bdf64ac9b8d0e38ab8706fcf9ee1d54e70a@95.217.35.186:60656,6e7e9341fba194988d448393b2d77464107385c5@65.108.199.222:22496,514497b0bf03a0620af9d2d3e6fe540aed0b3b21@65.21.132.27:48356,a0a165866cf5408ed26459ff91e3968807fb13dd@152.228.215.7:26656,4ff352af6db6e68fc6913e82589c4c8dbfc88f6c@35.76.185.2:26656,8e4d886e7c333e73cdf1f0271b05511a1866d515@65.109.49.163:56656,439d6746ec2ddeb03a4328e9ab1d0806e5d46ccd@34.252.21.196:26656,e8eec527c57c1231e5b1bb8aea358806bd313ff0@44.211.124.4:26656,325eaed0931fc7d743c6ee9b124bca334ff8dc2c@65.109.92.241:21216,be84f739a3581e0b37b4e06716e9e136bb5ab746@35.232.184.57:26656,c6a2c0ed97f3a7c61073b758191d7375aad56163@34.67.27.129:26656,6f811ea67bcf1275ef55e0535630af783f767344@95.214.53.178:26656,c7e5de7911802a8c7f80c046ad93152476898d56@202.61.194.254:36656,1797a6e3a45ea538dc669e296e5f76a3b510d101@65.109.29.150:26656,81fc7f83e9961790a279a1fbe3e2835cea032d0c@37.252.184.229:26656,3b25a50bf0fd8f5e776d2e17f4a0d75883bca7fb@65.108.227.42:26656,f15b2375cbfb2b9200096e311b8a1f703e7c2a68@149.102.153.162:26656,1959b9014fa0bd0eda445f84ef5dfe1195956cd0@77.37.176.99:26666,0da4f6242164ea9ce74bf6e8602c32d408140693@95.217.77.23:27656,9f78e5139d2b12e85800114691bc9feaf46208ae@65.108.0.165:15556,42e5c9923c06e2100a19814c2fffbbdea641032d@15.235.114.194:10456,3c0b3668f614b2b18fd4a5fc520553755c02d53c@65.108.130.189:26656,6b90bb94063007ff88c14585debd84ababd7d637@65.108.79.198:26766
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.cantod/config/config.toml
```

## Addrbook for Canto

Sometimes node operators will face peering issues with the rest of the network. Often it can be solved with a good seed node or a list of stable persistent peers.

However, when everything else fails, you can choose to trust our **`addrbook.json`** file to bootstrap your node's connection with the network.

Stop your node, download and replace your **`addrbook.json`** with the steps below, and restart your node.

```shell
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/canto/addrbook.json --inet4-only
mv addrbook.json ~/.cantod/config
```

## Create (or restore) a local key pair

Either create a new key pair, or restore an existing wallet for your validator:

```shell
# Create new keypair
cantod keys add WhisperNode

# Restore existing Canto wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
cantod keys add WhisperNode --recover

# Query the keystore for your public address
cantod keys show WhisperNode -a
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
cantod tx staking create-validator \
  --amount 1000000acanto \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(cantod tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --identity=""
  --fees 0acanto \
  --from <key-name>
```

The above transaction is just an example. There are many more flags that can be set to customize your validator, such as your validator website, or keybase.io ID, etc. To see a full list:

```shell
cantod tx staking create-validator --help
```

---

# Backup critical files

There are certain files that you need to back up to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.cantod/config/`:

- `priv_validator_key.json`
- `node_key.json`

It is recommended that you encrypt the backup of these files.
