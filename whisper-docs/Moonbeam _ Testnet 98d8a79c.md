Axelar Guide:

[Moonbeam | Axelar Network](https://docs.axelar.dev/validator/external-chains/moonbeam "Moonbeam | Axelar Network")

Moonbeam Original Guide:

[Run a Node on Moonbeam Using Systemd | Moonbeam Docs](https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/ "Run a Node on Moonbeam Using Systemd | Moonbeam Docs")

Block Explorer:

[Moonbase Alpha (DEV) Blockchain Explorer](https://moonbase.moonscan.io/ "Moonbase Alpha (DEV) Blockchain Explorer")

GitHub Repo:

<https://github.com/binance-chain/bsc/releases/download/v1.1.4/geth_linux>

---

## Install Moonbase Alpha

### Download compiled binary

Download the [latest release binary](https://github.com/PureStake/moonbeam/tags) from PureState. In this tutorial, we are using `v0.28.0`

```shell
https://github.com/PureStake/moonbeam/releases/download/v0.27.2/moonbeam

# verify shasum256
sha256sum moonbeam

# should output:
# 20466828dec7a3035e209402c2d85bf3190b08f5477c6d51fa02f8cf7e985412
```

### Create a service account and copy the binary

```shell
sudo mkdir /var/lib/alphanet-data
sudo mv ./moonbeam /var/lib/alphanet-data
sudo chown -R whispernode /var/lib/alphanet-data
```

### Create the systemd service file

After the installation of `moonbeam`, we are now ready to start the node, but to ensure it is running in the background and auto-restarts in case of a server failure, we will set up a service file using systemd.

Note: In the service file below, you need to replace `"YOUR-NODE-NAME"` and replace `50% RAM in MB` for 50% of the actual RAM your server has (Example: `--db-cache 16000` if your server has 32GB RAM).

If you are connecting to Testnet instead (Moonbase Alpha), you will also need to change the path to `/home/whispernode/alphanet-data/` and add `--chain alphanet`.

```shell
# create service
sudo nano /etc/systemd/system/moonbeam.service

# unit file
[Unit]
Description="Moonbase Alpha systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=whispernode
SyslogIdentifier=moonbeam
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/alphanet-data/moonbeam \
     --port 30333 \
     --rpc-port 9933 \
     --ws-port 9944 \
     --unsafe-rpc-external \
     --rpc-cors all \
     --execution wasm \
     --wasm-execution compiled \
     --pruning=10000 \
     --state-cache-size 0 \
     --db-cache 32000 \
     --base-path /var/lib/alphanet-data \
     --chain alphanet \
     --name "WhisperNode-0" \
     -- \
     --port 30334 \
     --rpc-port 9934 \
     --ws-port 9945 \
     --execution wasm \
     --pruning=10000 \
     --name="WhisperNode-0 (Embedded Relay)"

[Install]
WantedBy=multi-user.target
```

### Enable and start the moonbeam service

```shell
sudo systemctl daemon-reload
sudo systemctl enable moonbeam.service
sudo systemctl start moonbeam.service
```

If everything was set up correctly, your Moonbeam node should now be starting the process of synchronization. This will take several hours, depending on your hardware. To check the status of the running service or to follow the logs, use:

```shell
sudo systemctl status moonbeam.service
journalctl -u moonbeam.service -f
```

## Test your Moonbeam RPC connection

Once your `Moonbeam` node is fully synced, you can run a cURL request to see the status of your node:

```shell
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "eth_syncing", "params":[]}' localhost:9933
```

If the node is successfully synced, the output from above will print `{"jsonrpc":"2.0","result":false,"id":1}`

### EVM RPC endpoint URL

Axelar Network will be connecting to the EVM compatible `Moonbeam`, so your `rpc_addr` should be exposed in this format:

```shell
http://65.109.38.40:9933
```

---

## Update Moonbase Alpha

As Moonbeam development continues, upgrading your node software will sometimes be necessary. Node operators will be notified on our [Discord channel](https://discord.gg/PfpUATX) when upgrades are available and whether they are necessary (some client upgrades are optional). The upgrade process is straightforward and is the same for a full node or collator.

If you want to update your client, you can keep your existing chain data intact, and only update the binary by following these steps:

Stop the `systemd` service

```shell
sudo systemctl stop moonbeam.service
```

Remove the old binary file:

```shell
rm  /var/lib/alphanet-data/moonbeam
```

Get the latest version of Moonbeam from the [Moonbeam GitHub Release](https://github.com/PureStake/moonbeam/releases/) page

If you're using the release binary, update the version and run:

```shell
wget https://github.com/PureStake/moonbeam/releases/download/v0.27.2/moonbeam
```

If you want to compile the binary, please refer back to the [Compile the Binary](https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#compile-the-binary) instructions, making sure you `git checkout` to the latest version.

Move the binary to the data directory:

```shell
# If you used the release binary:
mv ./moonbeam /var/lib/alphanet-data

# Or if you compiled the binary:
mv ./target/release/moonbeam /var/lib/alphanet-data
```

Update permissions:

```shell
sudo chmod +x /var/lib/alphanet-data/moonbeam
sudo chown whispernode /var/lib/alphanet-data/moonbeam
```

Start your service:

```shell
systemctl start moonbeam.service
```

To check the status of the service and/or logs, you can refer to the commands from before.

---

## Purge Your Node

If you need a fresh instance of your Moonbeam node, you can purge your node by removing the associated data directory.

Depending on whether you used the release binary or compiled the binary yourself, the instructions for purging your chain data will slightly vary. If you compiled the binary yourself, you can skip ahead to the [Purge Compiled Binary](https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#purge-compiled-binary) section.

### Purge Release Binary

You'll first need to stop the systemd service:

```shell
sudo systemctl stop moonbeam
```

To purge your parachain and relay chain data, you can run the following command:

```shell
sudo rm -rf /var/lib/alphanet-data/*
```

To only remove the parachain data for a specific chain, you can run:

```shell
sudo rm -rf /var/lib/alphanet-data/chains/*
```

Similarly, to only remove the relay chain data, you can run:

```shell
sudo rm -rf /var/lib/alphanet-data/polkadot/*
```

Now that your chain data has been purged, you can start a new node with a fresh data directory. You can install the newest version by repeating the [Installation Instructions](https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#installation-instructions). Make sure you are using the latest tag available, which you can find on the [Moonbeam GitHub Release](https://github.com/PureStake/moonbeam/releases/) page.

## Note

On an as-needed basis, Moonbase Alpha might be purged and reset. In these instances, you will need to purge both the parachain data and the relay chain data. If a purge is required, node operators will be notified in advance (via our [Discord channel](https://discord.gg/PfpUATX)).

### Purge Compiled Binary

If you want to start a fresh instance of a node, there are a handful of `purge-chain` commands available to you which will remove your previous chain data as specified. The base command which will remove both the parachain and relay chain data is:

```shell
./target/release/moonbeam purge-chain
```

You can add the following flags to the above command if you want to specify what data should be purged:

- `--parachain` - only deletes the parachain database, keeping the relay chain data intact
- `--relaychain` - only deletes the relay chain database, keeping the parachain data intact

You can also specify a chain to be removed:

- `--chain` - specify the chain using either one of the predefined chains or a path to a file with the chainspec

To purge only your Moonbase Alpha parachain data, for example, you would run the following command:

```shell
./target/release/moonbeam purge-chain --parachain --chain alphanet
```

To specify a path to the chainspec for a development chain to be purged, you would run:

```shell
./target/release/moonbeam purge-chain --chain example-moonbeam-dev-service.json
```

For the complete list of available `purge-chain` commands, you can access the help menu by running:

```shell
./target/release/moonbeam purge-chain --help
```

Now that your chain data has been purged, you can start a new node with a fresh data directory. You can install the newest version by repeating the [Installation Instructions](https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#installation-instructions). Make sure you are using the latest tag available, which you can find on the [Moonbeam GitHub Release](https://github.com/PureStake/moonbeam/releases/) page.
