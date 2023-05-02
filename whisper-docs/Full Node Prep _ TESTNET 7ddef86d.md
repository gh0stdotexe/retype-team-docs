Original Guide:

[Node configuration | Axelar Network](https://docs.axelar.dev/node/config-node "Node configuration | Axelar Network")

Axelar Testnet Explorer:

[Axelarscan | Axelar Network Explorer](https://testnet.axelarscan.io/validator/axelarvaloper17jw3znsqdfjp95sgj2ws85gcznck5a66nl9q4s "Axelarscan | Axelar Network Explorer")

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

*For an Ubuntu LTS, we can use:*

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

*Unless you want to configure in a nonstandard way, then set these in the* `.zshrc` *in the user's home (i.e.,* *`~/`) folder.*

```shell
nano ~/.zshrc
```

*Add the "export Pathing" rules  at the bottom, and then save the file:*

```shell
# add export PATHS below
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin:$GOBIN
```

*After updating your* `~/.zshrc` *you will need to source it:*

```shell
source ~/.zshrc
```

# Build Daemon from source

New (Updated 2/4/22):

```shell
git clone https://github.com/axelarnetwork/axelarate-community
```

Scripts are used to install Binaries. Scripts are run with specific tags depending on your variables, take a look at the Scripts in GitHub before running the .sh file to determine what is needed.&#x20;

**Flags:**

|   | -h, --help Print this help and exit                                                  |
| - | ------------------------------------------------------------------------------------ |
|   | -v, --verbose Print script debug info                                                |
|   | -r, --reset-chain Reset all chain data (erases current state including secrets)      |
|   | -a, --axelar-core-version Version of axelar core to checkout                         |
|   | -d, --root-directory Directory for data. \[default: $HOME/.axelar\_testnet]          |
|   | -n, --network Core Network to connect to \[devnet\|testnet\|mainnet]                 |
|   | -t, --tendermint-key-path Path to tendermint key                                     |
|   | -m, --axelar-mnemonic-path Path to axelar mnemonic key                               |
|   | -e, --environment Environment to run in \[docker\|host] (host uses release binaries) |
|   | -c, --chain-id Axelard Chain ID \[default: axelar-testnet-lisbon]                    |
|   | -k, --node-moniker Node Moniker \[default: hostname]                                 |

*Before Running Scripts Axelar requires you to set the* `KEYRING_PASSWORD` *&* `TOFND` *PASSWORD for encryption. To do so do the following:*

```shell
cd ..
nano .bashrc
```

*Set* `PASSWORDS`*,* `chain-id`*, and alias accordingly in* `.bashrc` *file:*

```shell
export KEYRING_PASSWORD=mQaGaPBN49Up
export TOFND_PASSWORD=h3CuSG1072iK
export AXELARD_CHAIN_ID=axelar-testnet-lisbon-3

alias axelard=~/.axelar/bin/axelard-v0.14.1
```

*Source new* `.bashrc` *file & Change back to axelarate-community directory:*

```shell
source .bashrc
cd axelarate-community
```

*Change node **moniker**, put in the **external address*** *, an\d put in **RPC Nodes*** *in the  BASE*`  config.toml. This config.toml is copied over when the scripts are run for each service.  `

```shell
nano ~/axelarate-community/configuration/config.toml 
```

```shell
# This is a TOML config file.
# For more information, see https://github.com/toml-lang/toml

# NOTE: Any path below can be absolute (e.g. "/var/myawesomeapp/data") or
# relative to the home directory (e.g. "data"). The home directory is
# "$HOME/.tendermint" by default, but could be changed via $TMHOME env variable
# or --home cmd flag.

#######################################################################
###                   Main Base Config Options                      ###
#######################################################################

# TCP or UNIX socket address of the ABCI application,
# or the name of an ABCI application compiled in with the Tendermint binary
proxy_app = "tcp://127.0.0.1:26658"

# A custom human readable name for this node
moniker = "WhisperNode"

............

# Address to advertise to peers for them to dial
# If empty, will use the same port as the laddr,
# and will introspect on the listener or use UPnP
# to figure out the address. ip and port are required
# example: 159.89.10.97:26656
# external_address = ""
external_address = "[LOCAL IP]:26656"

.............

[axelar_bridge_evm]]
name = "Ethereum"
rpc_addr = "http://95.217.118.120:8545"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Avalanche"
rpc_addr = "http://23.92.74.122:9650/ext/bc/C/rpc"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Fantom"
rpc_addr = "http://135.181.223.196:18545"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Moonbeam"
rpc_addr = "http://135.181.216.48:30333"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Polygon"
rpc_addr = ""
start-with-bridge = false

```

*Change back to axelarate-community directory to run the script:*

```shell
cd axelarate-community
```

## Setup Scripts:

<br>

> **Mainnet**: R*un the following script to set up Axelar for mainnet:*

```shell
./scripts/node.sh -a v0.10.7 -n mainnet -d ~/.axelar -m ~/mnemonic -t ~/tendemint -e host -c axelar-dojo-1
```

<br>

> **Testnet**: *Run the following script to configure Axelar for testnet:*

```shell
./scripts/node.sh -a v0.10.7 -n testnet -d ~/.axelar -t ~/tendemint -e host -c axelar-testnet-lisbon-2
```

---

# Backup Mnemonics & Keys:

BACKUP and DELETE the validator account mnemonic: Validator mnemonic: /home/whispernode/.axelar/validator.txt

BACKUP but do NOT DELETE the Tendermint consensus key (this is needed on node restarts): Tendermint consensus key: /home/whispernode/.axelar/.core/config/priv\_validator\_key.json

<br>

*To confirm that the installation has succeeded and you're up and running, you can run:*

```shell
tail -f /home/whispernode/.axelar/logs/axelard.log
```

---

# Setting up VALD & TOFND

*In the validators directory run the script to launch tools with the necessary flags as described above:*

```shell
./scripts/validator-tools-host.sh -k WhisperNode -c axelar-testnet-lisbon-2 -d $HOME/.axelar -e host -a v0.13.0 -q v0.8.2
```

# BACKUP Broadcaster & TOFND Keys

After running the script you will get a printout like the following in which you will find the files for the  keys to back up:

```shell
Broadcaster address: axelar155ej5wl5cds3e9386kcv67u4m47ynvh0qwpgvs

# To follow tofnd execution, run:
tail -f /home/whispernode/.axelar/logs/tofnd.log

# To follow vald execution, run:
tail -f /home/whispernode/.axelar/logs/vald.log

# To stop tofnd, run:
kill -9 $(pgrep tofnd)

# To stop vald, run 
kill -9 $(pgrep -f "axelard vald-start")

SUCCESS

CHECK the logs to verify that the processes are running as expected

### BACKUP and DELETE the following mnemonics:

# Tofnd mnemonic: 
/home/whispernode/.axelar/.tofnd/import

# Broadcaster mnemonic: 
/home/whispernode/.axelar/broadcaster.txt
```

# Fund Validator Address and Broadcaster Address

To register the broadcaster and validator you will need funds in each wallet. You can get tokens from the team in the case of mainnet, or in the case of testnet use the following link:

[Axelar Testnet Faucet](https://faucet.testnet.axelar.dev/ "Axelar Testnet Faucet")

```shell
 axelard q bank balances (validator or broadcaster address) --home ~/.axelar/.core
```

# Register the Broadcaster Address & Create Validator

To register broadcaster you will need to add the validator key to the machine or sign offline from local machine. To recover using the mnemonic you can use:

```shell
axelard keys add validator --recover
```

\
Then you will run:&#x20;

```shell
axelard tx snapshot register-proxy axelar130gefctahcpwwu46hnccj5ms436z2u00vlfqzk --from validator --chain-id axelar-testnet-lisbon-3 --gas-prices 0.05uaxl
```

### Upgrade to a validator

> *Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.*

The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```shell
Register the Node:
```

```shell
axelard tx staking create-validator --yes --amount "2000000uaxl" --moniker "WhisperNode" --commission-rate="0.10" --commission-max-rate="0.20" --commission-max-change-rate="0.01" --min-self-delegation="1" --pubkey=$(axelard tendermint show-validator --home ~/.axelar/.core) --chain-id axelar-testnet-lisbon-3 --from validator --identity 9C7571030BEF5157 -b block 
```

<br>

Edit the node:

```shell
axelard tx staking edit-validator --details "WhisperNode operates robust, high up-time validators across multiple Cosmos-based blockchains. We are highly active participants in the communities we validate for as we believe this is a necessity for effective on-chain governance and staying up to date on all network issues." --chain-id axelar-testnet-lisbon --identity "9C7571030BEF5157" --from validator --fees 5000uaxl
```

```shell
axelard tx staking edit-validator --from validator --chain-id axelar-dojo-1 --moniker=" WhisperNode🤐" --website="https://REStake.WhisperNode.com/axelar" --details="WhisperNode operates robust, high up-time validators across the Cosmos ecosystem. Auto-compound your rewards with our REStake app. Join our Discord: https://discord.gg/4E5KZsRtjE / tweet @ https://twitter.com/WhisperNode 🐦" --security-contact="security@whispernode.com" --commission-rate="0.05" --gas auto --gas-adjustment 1.2 --node http://127.0.0.1:26657
```

# Register as a Chain Maintainer

General Command

```shell
axelard tx nexus register-chain-maintainer [network] --from broadcaster --node [http://IPOFFULLNODE:26657] --chain-id axelar-dojo-1
```

Specific Command&#x20;

```shell
axelard tx nexus register-chain-maintainer ethereum --from broadcaster --node http://23.88.3.237:26657 --chain-id axelar-dojo-1
```

Edit Validator

```shell
axelard tx staking edit-validator --from validator --chain-id axelar-dojo-1 --moniker=" WhisperNode🤐" --website="https://REStake.WhisperNode.com/axelar" --details="WhisperNode operates robust, high up-time validators across the Cosmos ecosystem. Auto-compound your rewards with our REStake app. Join our Discord: https://discord.gg/4E5KZsRtjE / tweet @ https://twitter.com/WhisperNode 🐦" --security-contact="security@whispernode.com" --commission-rate="0.05" --gas auto --gas-adjustment 1.2 --node http://127.0.0.1:26657
```

<br>
