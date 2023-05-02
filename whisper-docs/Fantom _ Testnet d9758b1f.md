### Axelar Guide:

[Fantom | Axelar Network](https://docs.axelar.dev/validator/external-chains/fantom "Fantom | Axelar Network")

GitHub Repo:

[GitHub - Fantom-foundation/go-opera: Opera blockchain...](https://github.com/Fantom-foundation/go-opera "GitHub - Fantom-foundation/go-opera: Opera blockchain protocol secured by the Lachesis consensus algorithm")

Snapshot Links:

[Snapshot download - Fantom](https://docs.fantom.foundation/staking/snapshot-download "Snapshot download - Fantom")

Block Explorer:

[TESTNET Fantom (FTM) Blockchain Explorer](https://testnet.ftmscan.com/ "TESTNET Fantom (FTM) Blockchain Explorer")

---

## Install required dependencies[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#install-required-dependencies "Direct link to heading")

In order to build the `go-opera`, you first need to install all the required dependencies.

### 1. Update and install build-essential[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#1-update-and-install-build-essential "Direct link to heading")

```shell
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y build-essential
```

### 2. Install golang[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#2-install-golang "Direct link to heading")

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

Unless you want to configure in a non-standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

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

## Install Go-Opera[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#install-go-opera "Direct link to heading")

### 1. Checkout and build go-opera[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#1-checkout-and-build-go-opera "Direct link to heading")

Use the commands below to clone the `go-opera` repository, checkout correct version and compile the binary.

```shell
git clone https://github.com/Fantom-foundation/go-opera.git
cd go-opera/
git checkout release/1.1.1-rc.2
make
```

### 2. Download the genesis file[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#2-download-the-genesis-file "Direct link to heading")

```shell
cd build/
wget https://download.fantom.network/testnet-6226-pruned-mpt.g
```

### 3. Create systemd service file

After installation of `go-opera`, we are now ready to start the process but in order to ensure it is running in the background and auto-restarts in case of a server failure, we will setup a service file with systemd.

📝

Note: In the service file below, you need to replace `$USER` and path to `opera`, depending on your system configuration.

```shell
# install service
sudo nano /etc/systemd/system/fantom.service

# add unit file
[Unit]
Description=Fantom Node
After=network.target

[Service]
User=whispernode
Type=simple
ExecStart=/home/whispernode/go-opera/build/opera --genesis /home/whispernode/go-opera/build/testnet-6226-pruned-mpt.g --identity WhisperNode --cache 8096 --http --http.addr 0.0.0.0 --http.corsdomain '*' --http.vhosts "*" --http.api "eth,net,web3" 
Restart=always
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

⚠️

If you would like to run a node on the Testnet instead, you need to replace `--genesis /root/go-opera/build/testnet.g` in the configuration above.

### 4. Enable and start the fantom service

```shell
sudo systemctl enable fantom
sudo systemctl daemon-reload
sudo systemctl start fantom
```

If everything was set up correctly, your Fantom node should now be starting the process of synchronization. This will take several hours, depending on your hardware. To check the status of the running service or to follow the logs, you can use:

```shell
sudo systemctl status fantom
sudo journalctl -u fantom -f
```

Your node will now start to synchronize with the network. It will take numerous hours before the node is fully synced.

---

## Test your Fantom RPC connection[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#test-your-fantom-rpc-connection "Direct link to heading")

Once your node is fully synced, you can run a cURL request to see the status of your node:

```shell
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "eth_syncing", "params":[]}' localhost:18545
```

If the node is successfully synced, the output from above will print `{"jsonrpc":"2.0","id":1,"result":false}`

### EVM RPC endpoint URL[​](https://docs.axelar.dev/roles/validator/external-chains/fantom#evm-rpc-endpoint-url "Direct link to heading")

In order for Axelar Network to connect to your Fantom node, your `rpc_addr` should be exposed in this format:

```shell
http://192.168.192.168:18545
```

<br>
