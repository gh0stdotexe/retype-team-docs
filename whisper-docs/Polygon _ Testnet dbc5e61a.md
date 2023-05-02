Original guide:

[Polygon | Axelar Network](https://docs.axelar.dev/validator/external-chains/polygon "Polygon | Axelar Network")

Github:

[maticnetwork/heimdall: Validator node for Matic PoS layer](https://github.com/maticnetwork/heimdall)

Snapshots:

<https://snapshots.matic.today>

Block Explorer:

[TESTNET Polygon (MATIC) Blockchain Explorer](https://mumbai.polygonscan.com/ "TESTNET Polygon (MATIC) Blockchain Explorer")

---

# Install required dependencies

In order to build the `polygon` node, you first need to install all of the required dependencies.

### 1. Update and install build-essential

```shell
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y build-essential
```

### 2. Install golang

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

## Install the Polygon node

Polygon node consists of 2 layers, Heimdall and Bor. Heimdall is a fork of tendermint and runs in parallel to the Ethereum network, monitoring contracts, and Bor is a fork of go-Ethereum and producing blocks shuffled by Heimdall nodes. You need to install and run both binaries in the correct order, as explained in the following steps.

### 1. Install Heimdall

Please make sure you checkout the [latest release tag](https://github.com/maticnetwork/heimdall/tags), depending on the network (Mainnet/Testnet). In this tutorial, we are using `v0.2.12`

```shell
cd ~/
git clone https://github.com/maticnetwork/heimdall
cd heimdall
git checkout v0.2.12
make install

# Verify the correct version
heimdalld version --long
```

### 2. Install Bor

Please make sure you checkout the [latest release tag](https://github.com/maticnetwork/bor/tags). In this tutorial, we are using `v0.2.17`

```shell
cd ~/
git clone https://github.com/maticnetwork/bor
cd bor
git checkout v0.2.17
make bor-all
sudo ln -nfs ~/bor/build/bin/bor /usr/bin/bor
sudo ln -nfs ~/bor/build/bin/bootnode /usr/bin/bootnode

# Verify the correct version
bor version
```

## Setup and configure node

### 1. Setup launch directory

Replace the `<network-name>` below with the network you are joining. Available networks: `mainnet-v1` and `testnet-v4`.

```shell
cd ~/
git clone https://github.com/maticnetwork/launch

mkdir -p node
cp -rf launch/testnet-v4/sentry/sentry/* ~/node
```

### 2. Setup network directories

```shell
# Heimdall
cd ~/node/heimdall
bash setup.sh

# Bor
cd ~/node/bor
bash setup.sh
```

### 3. Setup service files

Again, replace the `<network-name>` below with the network you are joining.

```shell
# Download service file
cd ~/node
wget https://raw.githubusercontent.com/maticnetwork/launch/master/testnet-v4/service.sh

# Generate Metadata
sudo mkdir -p /etc/matic
sudo chmod -R 777 /etc/matic/
touch /etc/matic/metadata

# Generate service file and copy them into systemd directory
cd ~/node
bash service.sh
sudo cp *.service /etc/systemd/system/
```

### 4. Setup config files

Open the `~/.heimdalld/config/config.toml` and edit the following flags:

```shell
nano ~/.heimdalld/config/config.toml

moniker=WhisperNode

#Testnet:
seeds="4cd60c1d76e44b05f7dfd8bab3f447b119e87042@54.147.31.250:26656,b18bbe1f3d8576f4b73d9b18976e71c65e839149@34.226.134.117:26656"

Change the value of Pex to true
Set the max_open_connections value to 100
```

Open the `~/.heimdalld/config/heimdall-config.toml` and edit:

```shell
nano ~/.heimdalld/config/heimdall-config.toml

eth_rpc_url = "http://65.108.238.47:8545"
```

Open the `~/node/bor/start.sh` and add/change the following flags to start parameters:

```shell
--http --http.addr '0.0.0.0' \

#Testnet:
--bootnodes "enode://320553cda00dfc003f499a3ce9598029f364fbb3ed1222fdc20a94d97dcc4d8ba0cd0bfa996579dcc6d17a534741fb0a5da303a90579431259150de66b597251@54.147.31.250:30303"
```

To enable Archive mode you can also add the following flags in the start.sh file

```shell
--gcmode 'archive' \
--ws --ws.port 8546 --ws.addr 0.0.0.0 --ws.origins '*' \
```

### 5. Download maintained snapshots

Syncing Heimdall and Bor services can take several days to sync fully. Alternatively, you can use snapshots to reduce the sync time to a few hours. If you wish to sync the node from the start, then you can skip this step.

To use the snapshots, please visit [Polygon Chains Snapshots](https://snapshots.matic.today/) and download the latest available snapshot for Heimdall and Bor. Replace the `snapshot-link` below with the full path to the snapshot of the network you're joining.

```shell
wget https://matic-blockchain-snapshots.s3-accelerate.amazonaws.com/matic-mumbai/heimdall-snapshot-2022-07-26.tar.gz -O - | tar -xzf - -C ~/.heimdalld/data/
wget https://matic-blockchain-snapshots.s3-accelerate.amazonaws.com/matic-mumbai/bor-fullnode-snapshot-2022-07-26.tar.gz -O - | tar -xzf - -C ~/.bor/data/bor/chaindata
# If needed, change the path depending on your server configuration.
```

## Start the Polygon services

After completing the previous steps, your node should be configured and ready to launch with the previously created service files.

### 1. Start Heimdalld

```shell
sudo service heimdalld start
sudo service heimdalld-rest-server start
```

Important: You need to wait for Heimdall node to fully sync with the network before starting the Bor service!

You can check the status of `heimdalld` service or follow the logs with:

```shell
sudo service heimdalld status
journalctl -u heimdalld.service -f
```

### 2. Start Bor

Once `heimdalld` is synced with the [latest block height](https://wallet.polygon.technology/staking/), then you can start the `bor` service file:

```shell
sudo service bor start

# Check status and logs
sudo service bor status
journalctl -u bor.service -f
```

## Test your Polygon RPC connection

Once your `Bor` node is fully synced, you can run a cURL request to see the status of your node:

```shell
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "eth_syncing", "params":[]}' localhost:8545
```

If the node is successfully synced, the output from above will print `{"jsonrpc":"2.0","id":1,"result":false}`

### EVM RPC endpoint URL

Axelar Network will be connecting to the EVM-compatible blockchain `Bor`, so your `rpc_addr` should be exposed in this format:

```shell
http://65.108.130.87:8545
```

<br>
