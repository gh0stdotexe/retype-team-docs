---

*The original guide can be found here:*

[nomic/README.md at develop · nomic-io/nomic](https://github.com/nomic-io/nomic/blob/develop/README.md "nomic/README.md at develop · nomic-io/nomic")

## Staking UI:

[https://app.nomic.io](https://app.nomic.io/ "Nomic Bitcoin Bridge")

Block Explorer:

<https://nomic.zenscan.io/validators.php>

---

# Install pre-requisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y

# install toolchain and ensure accurate time synchronization
sudo apt-get install make build-essential gcc git jq chrony -y
```

# Install Rust Nightly

<https://nomic.zenscan.io/validators.php>

For an Ubuntu LTS, we can use:

```shell
# install rustup if you haven't already
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# nomic currently requires rust nightly
rustup default nightly

# install required dependencies (ubuntu)
sudo apt install build-essential libssl-dev pkg-config clang -y

# clone
git clone https://github.com/nomic-io/nomic.git nomic && cd nomic
cargo install --locked --path .
```

---

# Initialize and configure your node

### Run your node

```shell
nomic start
```

**IMPORTANT NOTE:** Carefully double-check all the fields, since you will not be able to modify the `commission_max` or `commission_max_change` after declaring. If you make a mistake, you will have to declare a new validator instead.

- The `validator_consensus_key` field is the base64 pubkey `value` field found under `"validator_info"` in the output of <http://localhost:26657/status>.
- The `identity` field is the 64-bit hex key suffix found on your Keybase profile, used to get your profile picture in wallets and block explorers.

For example:

```shell
nomic declare \
  ohFOw5u9LGq1ZRMTYZD1Y/WrFtg7xfyBaEB4lSgfeC8= \
  100000 \
  0.042 \
  0.1 \
  0.01 \
  100000 \
  "Foo's Validator" \
  "https://foovalidator.com" \
  37AA68F6AA20B7A8 \
  "Please delegate to me!"
```

Visit <https://app.nomic.io> to claim the airdrop if you are an eligible ATOM holder/staker so that you can delegate more NOM to yourself.

Thanks for participating in the Nomic network! We'll update the network often, so stay tuned in [Discord](https://discord.gg/jH7U2NRJKn) for updates.

### Create Unit File for Nomic Service

###

Next, we'll create a service file so that our `nomic` service runs in the background + restarts automatically in the event of a failure/node restart / etc.

```shell
sudo nano /etc/systemd/system/nomic.service

# unit file
[Unit]
Description=Nomic daemon
After=network-online.target

[Service]
User=whispernode
ExecStart=/home/whispernode/.cargo/bin/nomic start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Then, we'll enable our newly created `nomic` service:

```shell
sudo systemctl daemon-reload
sudo systemctl enable nomic
sudo systemctl start nomic
```

Optimize Node:

Now, we'll run some typical node optimizations:

```shell
nano ~/.nomic-stakenet-3/tendermint/config/config.toml

====================
# update inbound + outbound peers

# Maximum number of inbound peers
max_num_inbound_peers = 100

# Maximum number of outbound peers to connect to, excluding persistent peers
max_num_outbound_peers = 80


====================
# update empty block creation
create_empty_blocks = false


====================
# set indexer = null
indexer = "null"
```

## Run your Bitcoin signer

The funds in the Bitcoin bridge are held in a large multisig controlled by the Nomic validators. If you are a validator with a significant amount of voting power, it is essential that you run a signer.

You can run the signer with:

```shell
nomic signer
```

This will automatically generate a Bitcoin extended private key and store it at `~/.nomic-stakenet-3/signer/xpriv`. It will also prompt you to submit your public key to the network, so you can be added to the multisig. **KEEP THIS KEY SAFE—similar** to your validator's private key, it is important to be mindful of this key so that it is never lost or stolen.

Next, we'll create a `systemd` service for `nomic-signer` so that it always runs / restarts in the event of a failure or server crash.

```shell
# create systemd service
sudo nano /etc/systemd/system/nomic-signer.service

# paste unit file
[Unit]
Description=Nomic Signer
After=nomic.service
Requires=nomic.service

[Service]
User=whispernode
WorkingDirectory=/home/whispernode/nomic
ExecStart=/home/whispernode/.cargo/bin/nomic signer
Restart=always
RestartSec=5
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

Leave this process running, it will automatically sign Bitcoin transactions that the network wants to create.

In the future, we hope for the community to come up with alternative types of signers that provide extra security, e.g., airgapping keys, using HSMs, or prompting the user for an encryption key.

## (Optional) Run a relayer

Relayer nodes carry data between the Bitcoin blockchain and the Nomic blockchain. You can help support the network's health by running a Bitcoin node alongside your Nomic node and the relayer process.

### Sync a Bitcoin node

Download Bitcoin Core: <https://bitcoin.org/en/download>

Run it with:

```shell
bitcoind -server -rpcuser=satoshi -rpcpassword=nakamoto
```

(The RPC server only listens on localhost, so the user and password are not critically important.)

Next, we'll create a `systemd` service for `bitcoind`:

```shell
# create serviceD5740592 num_txs=0
Aug 29 06:49:00 master-val nomic[74621]: I[2022-08-29|06:49:00.824] executed block                               module=state height=1232883 num_valid_txs=0 num_invalid_txs=0
Aug 29 06:49:00 master-val nomic[74621]: I[2022-08-29|06:49:00.867] committed state                              module=state height=1232883 num_txs=0 app_hash=102ACCC11579A93E2CDEEDB1342E9EC92C7D53F9D22523213B65B13ECE24EFEF
Aug 29 06:49:00 master-val nomic[74621]: I[2022-08-29|06:49:00.867] indexed block                                module=txindex height=1232883
Aug 29 06:49:03 master-val nomic[74621]: I[2022-08-29|06:49:03.802] Timed out                                    module=consensus dur=2.933357025s height=1232884 round=0 step=RoundStepNewHeight
Aug 29 06:49:04 master-val nomic[74621]: I[2022-08-29|06:49:04.135
sudo nano /etc/systemd/system/bitcoin.service

# paste into service file
[Unit]
Description=Bitcoin testnet
After=network-online.target

[Service]
User=whispernode
WorkingDirectory=/home/whispernode/nomic
ExecStart=/home/whispernode/bitcoin-22.0/bin/bitcoind -testnet -server -rpcuser=satoshi -rpcpassword=nakamoto -prune=2000
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

**NOTE:** To save on disk space, you may want to configure your Bitcoin node to prune block storage. For instance, add `-prune=5000` to only keep a maximum of 5000 MB of blocks. You may also want to use the `-daemon` option to keep the node running in the background.

### ii. Run the relayer process

```shell
nomic relayer --rpc-port=8332 --rpc-user=satoshi --rpc-pass=nakamoto
```

Leave this running - the relayer will constantly scan the Bitcoin and Nomic chains and broadcast relevant data.

We'll create a service file for this process as well:

```shell
# create service
sudo nano /etc/systemd/system/nomic-relayer.service

# paste into service file
[Unit]
Description=Nomic Relayer
After=network-online.target nomic.service nomic-signer.service bitcoin.service

[Service]
User=whispernode
WorkingDirectory=/home/whispernode/nomic
ExecStart=/home/whispernode/.cargo/bin/nomic relayer --rpc-port=18332 --rpc-user=satoshi --rpc-pass=nakamoto
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

The relayer will also create a server that listens on port 9000 for clients to announce their deposit addresses. To help make the network more reliable, if you run a relayer please open this port and let us know your node's address in Discord or a Github issue so we can have clients use your node. If you're going to make this service public, putting the server behind an HTTP reverse proxy is recommended for extra safety.
