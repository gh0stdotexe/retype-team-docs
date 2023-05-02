---

Original Guide:

[How To Join Secret Network as a Full Node | Secret Network](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#installation "How To Join Secret Network as a Full Node | Secret Network")

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
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm -rf go1.20.linux-amd64.tar.gz
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

## Installation

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_0-step-up-sgx-on-your-local-machine)0. Step up SGX on your local machine

See instructions for [**setup**](https://docs.scrt.network/secret-network-documentation/node-runners/node-setup/install-sgx "Install SGX - Secret Network Documentation") and [**verification**](https://docs.scrt.network/node-guides/verify-sgx.html). See [**registration**](https://docs.scrt.network/node-guides/registration.html) if you'd like a more comprehensive overview on what's happening in these steps.

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_1-download-the-secret-network-package-installer-for-debian-ubuntu)1. Download the Secret Network package installer for Debian/Ubuntu:

```shell
wget "https://github.com/scrtlabs/SecretNetwork/releases/download/v1.6.0/secretnetwork_1.6.0_mainnet_goleveldb_amd64.deb"
# check the hash of the downloaded binary
echo "ce9ba85d346fa460ed3fc98871f2a254b269fafa835fc555c9184f6405d8c80a secretnetwork_1.6.0_mainnet_goleveldb_amd64.deb" | sha256sum --check
```

([**How to verify releases**](https://docs.scrt.network/verify-releases.html))

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_2-install-the-package)2. Install the package:

Note: if you are upgrading from `v1.2.0`, it may say `secret-node` is downgrading to version `0`. Ignore it.

```shell
sudo apt install -y ./secretnetwork_1.6.0_mainnet_*_amd64.deb

# verify installation
secretd version
# 1.6.0
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_3-initialize-your-installation-of-the-secret-network)3. Install SGX

<https://docs.scrt.network/secret-network-documentation/node-runners/node-setup/install-sgx>\
\
3a. Download the SGX install script.

```shell
wget https://raw.githubusercontent.com/SecretFoundation/docs/main/docs/node-guides/sgx
```

3b. Execute Script

```shell
sudo bash sgx
```

3c. Init Enclave

```shell
secretd init-enclave
```

### #4 Initialize your installation of the Secret Network.

Choose a **moniker** for yourself, and replace `<MONIKER>` with your moniker below. This moniker will serve as your public nickname in the network.

```shell
secretd init <moniker> --chain-id secret-4
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_4-download-a-copy-of-the-genesis-block-file-genesis-json)5. Download a copy of the Genesis Block file: genesis.json

```shell
wget -O ~/.secretd/config/genesis.json "https://github.com/scrtlabs/SecretNetwork/releases/download/v1.2.0/genesis.json"
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_5-validate-the-checksum-for-the-genesis-json-file-you-have-just-downloaded-in-the-previous-step)6. Validate the checksum for the genesis.json file you have just downloaded in the previous step:

```shell
echo "759e1b6761c14fb448bf4b515ca297ab382855b20bae2af88a7bdd82eb1f44b9 $HOME/.secretd/config/genesis.json" | sha256sum --check
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_6-validate-that-the-genesis-json-is-a-valid-genesis-file)7. Validate that the genesis.json is a valid genesis file:

```shell
secretd validate-genesis
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_7-the-rest-of-the-commands-should-be-run-from-the-home-folder-home-your-username)8. The rest of the commands should be run from the home folder (/home/\<your\_username>)

```shell
cd ~
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_8-initialize-secret-enclave)9. Initialize secret enclave

You can choose between two ways, **8a (automatic)** or **8b (manual)**:

**Note:** if this machine has been registered before, and have the following files:

```shell
/home/user/.sgx_secrets/
├── consensus_seed.sealed
└── new_node_seed_exchange_keypair.sealed
```

you can move them to `/opt/secret/.sgx_secrets` and skip to [**step 16**](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#16-add-persistent-peers-to-your-configuration-file) (if not working, try registering anyway).

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_8a-currently-broken-skip-to-8b-initialize-secret-enclave-automatic-registration-experimental)9a. CURRENTLY BROKEN - SKIP TO 8b. Initialize secret enclave - Automatic Registration (EXPERIMENTAL)

- **Note:** Make sure SGX is running or this step might fail.

Make sure the directory `/opt/secret/.sgx_secrets` exists:

```shell
mkdir -p /opt/secret/.sgx_secrets
```

Create env variables:

```shell
export SCRT_ENCLAVE_DIR=/usr/lib
export SCRT_SGX_STORAGE=/opt/secret/.sgx_secrets
```

Register:

```shell
secretd auto-register

# If you have already registered previously, or you wish to overwrite 
# an existing key you can force registration with the command

secretd auto-register --reset
```

**If this step was successful, you can skip straight to** [**step 16**](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#16-add-persistent-peers-to-your-configuration-file)

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_8b-initialize-secret-enclave-manual-registration-legacy)9b. Initialize secret enclave - Manual Registration (legacy)

Make sure the directory `/opt/secret/.sgx_secrets/` exists:

```shell
mkdir -p /opt/secret/.sgx_secrets/
```

Make sure SGX is running or this step might fail.

```shell
secretd init-enclave --reset
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_9-check-that-initialization-was-successful)10. Check that initialization was successful

Attestation certificate should have been created by the previous step

```shell
ls -lh /opt/secret/.sgx_secrets/attestation_cert.der
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_10-check-your-certificate-is-valid)11. Check your certificate is valid

Should print your 64 character registration key if it was successful.

```shell
PUBLIC_KEY=$(secretd parse /opt/secret/.sgx_secrets/attestation_cert.der  2> /dev/null | cut -c 3-)
echo $PUBLIC_KEY
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_11-config-secretcli-to-point-to-a-working-node-and-import-a-key-with-some-scrt)12. Config secretd, to point to a working node and import a key with some SCRT

The steps using `secretcli` can be run on any machine, they don't need to be on the full node itself. We'll refer to the machine where you are using `secretcli` as the "CLI machine" below.

To run the steps with `secretcli` on another machine, [**set up the CLI**](https://docs.scrt.network/cli/install-cli.html) there.

Configure `secretcli`. Initially you'll be using the bootstrap node, as you'll need to connect to a running node and your own node is not running yet.

```shell
secretd config chain-id $(curl https://chains.cosmos.directory/secretnetwork/chain.json | jq '.chain_id' -r)
secretd config node http://172.241.26.30:26657
secretd config output json
```

Set up a key. Make sure you back up the mnemonic and the keyring password.

```shell
secretd keys add WhisperNode
```

This will output your address, a 45 character-string starting with `secret1...`. Then you can fund it with some SCRT.

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_12-register-your-node-on-chain)13. Register your node on-chain

Run this step on the CLI machine. If you're using a different CLI machine than the full node, copy `/opt/secret/.sgx_secrets/attestation_cert.der` from the full node to the CLI machine.

```shell
secretd tx register auth /opt/secret/.sgx_secrets/attestation_cert.der -y --from WhisperNode --node http://172.241.26.30:26657
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_13-pull-check-your-node-s-encrypted-seed-from-the-network)14. Pull & check your node's encrypted seed from the network

Run this step on the CLI machine.

```shell
SEED=$(secretd query register seed $PUBLIC_KEY | cut -c 3-)
echo $SEED
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_14-get-additional-network-parameters)15. Get additional network parameters

Run this step on the CLI machine.

These are necessary to configure the node before it starts.

```shell
secretcli query register secret-network-params
ls -lh ./io-master-cert.der ./node-master-cert.der
```

If you're using a different CLI machine than the validator node, copy `node-master-cert.der` from the CLI machine to the validator node.

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_15-configure-your-secret-node)16. Configure your secret node

From here on, run commands on the full node again.

```shell
mkdir -p ~/.secretd/.node
secretd configure-secret node-master-cert.der $SEED
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_16-listen-for-incoming-rpc-requests-so-that-light-nodes-can-connect-to-you)17. Optimization

In order to be able to handle NFT minting and other Secret Contract-heavy operations, it's recommended to update your SGX memory enclave cache:

```shell
sed -i.bak -e "s/^contract-memory-enclave-cache-size *=.*/contract-memory-enclave-cache-size = \"15\"/" ~/.secretd/config/app.toml
```

### Also checkout this document by \[ block pane ] for fine tuning your machine for better uptime: <https://gist.github.com/blockpane/40bc6b64caa48fdaff3b0760acb51eaa>

\
[#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_17-enable-secret-node-as-a-system-service)18. Set Minimum Gas:

```shell
perl -i -pe 's/^minimum-gas-prices = .+?$/minimum-gas-prices = "0.0125uscrt"/' ~/.secretd/config/app.toml
```

\#19 Enable Secret Node and Start Service

Note that the `secret-node` system file is created in a previous step.

```shell
sudo systemctl enable secret-node && sudo systemctl start secret-node
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_19-if-everything-above-worked-correctly-the-following-command-will-show-your-node-streaming-blocks-this-is-for-debugging-purposes-only-kill-this-command-anytime-with-ctrl-c)20. If everything above worked correctly, the following command will show your node streaming blocks (this is for debugging purposes only, kill this command anytime with Ctrl-C):

```shell
journalctl -f -u secret-node
```

```shell
-- Logs begin at Mon 2020-02-10 16:41:59 UTC. --
Nov 09 11:16:31 scrt-node-01 secretd[619529]: 11:16AM INF indexed block height=12 module=txindex
Nov 09 11:16:35 scrt-node-01 secretd[619529]: 11:16AM INF Ensure peers module=pex numDialing=0 numInPeers=0 numOutPeers=0 numToDial=10
Nov 09 11:16:35 scrt-node-01 secretd[619529]: 11:16AM INF No addresses to dial. Falling back to seeds module=pex
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF Timed out dur=4983.86819 height=13 module=consensus round=0 step=1
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4","parts":{"hash":"58F6211ED5D6795E2AE4D3B9DBB1280AD92B2EE4EEBAA2910F707C104258D2A0","total":1}},"height":13,"pol_round":-1,"round":0,"signature":"eHY9dH8dG5hElNEGbw1U5rWqPp7nXC/VvOlAbF4DeUQu/+q7xv5nmc0ULljGEQR8G9fhHaMQuKjgrxP2KsGICg==","timestamp":"2021-11-09T11:16:36.7744083Z"}
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF received complete proposal block hash=0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4 height=13 module=consensus
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF finalizing commit of block hash=0AF9693538AB0C753A7EA16CB618C5D988CD7DC01D63742DC4795606D10F0CA4 height=13 module=consensus num_txs=0 root=E4968C9B525DADA22A346D5E158C648BC561EEC351F402A611B9DA2706FD8267
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF minted coins from module account amount=6268801uscrt from=mint module=x/bank
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF executed block height=13 module=state num_invalid_txs=0 num_valid_txs=0
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF commit synced commit=436F6D6D697449447B5B373520353520323020352032342031312032333820353320383720313137203133372031323020313638203234302035302032323020353720343520363620313832203138392032333920393920323439203736203338203131322035342032332033203233362034375D3A447D
Nov 09 11:16:36 scrt-node-01 secretd[619529]: 11:16AM INF committed state app_hash=4B371405180BEE3557758978A8F032DC392D42B6BDEF63F94C2670361703EC2F height=13 module=state num_txs=0
```

You are now a full node. 🎉

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_21-get-your-node-id-with)21. Get your node ID with:

```shell
secretd tendermint show-node-id
```

Be sure to point your CLI to your running node instead of the bootstrap node

```shell
secretcli config node tcp://localhost:26657
```

If someone wants to add you as a peer, have them add the above address to their `persistent_peers` in their `~/.secretd/config/config.toml`.

And if someone wants to use your node from their `secretcli` then have them run:

```shell
secretcli config chain-id secret-4
secretcli config output json
secretcli config node tcp://<your-public-ip>:26657
```

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_22-optional-make-your-full-node-into-a-validator)22. Optional: make your full node into a validator

To turn your full node into a validator, see [**Joining Mainnet as a Validator**](https://docs.scrt.network/node-guides/join-validator-mainnet.html).

### [#](https://docs.scrt.network/node-guides/run-full-node-mainnet.html#_23-optional-state-sync)23. Optional: State Sync

You can skip syncing from scratch or download a snapshot by [**State Syncing**](https://docs.scrt.network/node-guides/state-sync.html#mainnet-state-sync) to the current block.

23a. Configure app.toml, set pruning, disable iav-disable-fastnode, set snapshot intervals:

```shell
nano ~/.secretd/config/app.toml


pruning = "custom"

# These are applied if and only if the pruning strategy is custom.
pruning-keep-recent = "113"
pruning-keep-every = "0"
pruning-interval = "61"

# IAVLDisableFastNode enables or disables the fast node feature of IAVL. 
# Default is true.
iavl-disable-fastnode = true 

# snapshot-interval specifies the block interval at which local state sync snapshots are
# taken (0 to disable). Must be a multiple of pruning-keep-every.
snapshot-interval = 1000

# snapshot-keep-recent specifies the number of recent snapshots to keep and serve (0 to keep all).
snapshot-keep-recent = 5


```

23b. Assign RPC for snapshot data:

```shell
SNAP_RPC="https://scrt-rpc.agoranodes.com:443"
```

23c.&#x20;

```shell
BLOCK_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height | awk '{print $1 - ($1 % 1000)}'); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $BLOCK_HEIGHT $TRUST_HASH

# output should be similar to:
# 3506000 FCB54D74A4A33F8C1CC18A7240D76D87CB192A89C17837C4DB6C6140612DDFEB74A4A33F8C1CC18A7240D76D87CB192A89C17837C4DB6C6140612DDFEB
```

23d. Set Variables in Config.toml

```shell
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"https://rpc.cosmos.directory:443/secretnetwork,https://rpc.cosmos.directory:443/secretnetwork\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.secretd/config/config.toml
```

24e. Reset Database and Stop node

```shell
sudo systemctl stop secret-node && secretd tendermint unsafe-reset-all --home ~/.secretd/ --keep-addr-book
```

\
24f. Double check rpc nodes in config.toml, and set fast sync to true

```shell
nano ~/.secretd/config/config.toml

# If this node is many blocks behind the tip of the chain, FastSync
# allows them to catchup quickly by downloading blocks in parallel
# and verifying their commits
fast_sync = true

rpc_servers = "https://scrt-rpc.agoranodes.com:443,https://rpc.spartanapi.dev:443"
```

24g. Increase temp storage for chunk downloads

```shell
sudo umount -l /tmp && sudo mount -t tmpfs -o size=8G,mode=1777 overflow /tmp
```

&#x20;24h. Restart and Check Logs:

```shell
sudo systemctl restart secret-node && journalctl -fu secret-node
```

<br>
