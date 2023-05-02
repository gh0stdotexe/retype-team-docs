Original Guide:

[cheqd-node/deb-package-install.md at main · cheqd/cheqd-node](https://github.com/cheqd/cheqd-node/blob/main/docs/setup-and-configure/debian/deb-package-install.md "cheqd-node/deb-package-install.md at main · cheqd/cheqd-node")

Snapshot Download (official/team):

[cheqd Snapshots](https://snapshots.cheqd.net/#mainnet/ "cheqd Snapshots")

Notional Snapshots:

[Index of /](https://snapshot.notional.ventures/cheqd/ "Index of /")

# Install pre-requisites

```shell
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y
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

---

## Download and prepare the binary for the upgrade

Download the [v0.6](https://github.com/cheqd/cheqd-node/releases/tag/0.6.9)wget -O $HOME/.cheqdnode/config/genesis.json <https://raw.githubusercontent.com/cheqd/cheqd-node/main/persistent_chains/mainnet/genesis.json>[.9 binary from Github](https://github.com/cheqd/cheqd-node/releases/tag/0.6.9). Make sure you download the correct binary for your system architecture (in the example below, we've used Linux x86\_64):

```shell
wget -c https://github.com/cheqd/cheqd-node/releases/download/0.6.9/cheqd-noded-0.6.9-Linux-x86_64.tar.gz
```

### 4. Extract the new binary

Our binary release has two versions: for x86\_64 systems, as well as for Linux ARM systems. Extract the correct binary for your architecture. (Example below is for x86\_64.)

```shell
cd ~
mkdir -p extract
tar -xvf cheqd-noded-0.6.9-Linux-x86_64.tar.gz -C extract
```

The binary will be extracted to the extract (or other specified) folder above. This contains the binary and the license/README file. By default, though, the file permissions won't be correct, which you can fix by:

```shell
cd ~/extract/cheqd-noded-0.6.9-Linux-x86_64
sudo chmod +x cheqd-noded
```

The last command should say the file ownership is `cheqd:cheqd`.

### Replace the existing binary with the patched release

If you've got a standalone release, your binary will typically be in `/usr/bin`

```shell
mv ~/extract/cheqd-noded-0.6.9-Linux-x86_64.tar.gz/cheqd-noded ~/go/bin/cheqd-noded
```

### Confirm you have the correct version installed

This should now report as `0.6.9` (the patched version)

```shell
cheqd-noded version
0.6.9
```

1. **Initialize the node configuration files**

   The “moniker” for your node is a “friendly” name that will be available to peers on the network in their address book and allows easy searching of a peer's address book.
   ```shell
   cheqd-noded init D8CCD8919980FBD113ED179F2CD004C8 --chain-id cheqd-mainnet-1
   ```
2. **Download the genesis file for a persistent chain, such as the cheqd testnet**

   Download the `genesis.json` file for the relevant [persistent chain](https://github.com/cheqd/cheqd-node/tree/main/persistent_chains/) and put it in the `$HOME/.cheqdnode/config` directory.

   For cheqd mainnet:
   ```shell
   wget -O $HOME/.cheqdnode/config/genesis.json https://raw.githubusercontent.com/cheqd/cheqd-node/main/persistent_chains/mainnet/genesis.json
   ```
3. **Define the seed configuration for populating the list of peers known by a node**

   Update `seeds` with a comma separated list of seed node addresses specified in `seeds.txt` for the relevant [persistent chain](https://github.com/cheqd/cheqd-node/tree/main/persistent_chains/).

   For cheqd mainnet, set the `SEEDS` environment variable:
   ```shell
   SEEDS=$(wget -qO- https://raw.githubusercontent.com/cheqd/cheqd-node/main/persistent_chains/mainnet/seeds.txt)
   ```
   After the `SEEDS` variable is defined, pass the values to the `cheqd-noded configure` tool to set it in the configuration file.
   ```shell
   echo $SEEDS
   # Comma separated list should be printed

   cheqd-noded configure p2p seeds "$SEEDS"
   ```
4. **Set gas prices accepted by the node**

   Update `minimum-gas-prices` parameter if you want to use custom value. The default is `25ncheq`.
   ```shell
   cheqd-noded configure min-gas-prices "25ncheq"
   ```
5. **Turn off empty block creation**

   By default, the underlying Tendermint consensus creates blocks even when there are no transactions (“empty blocks”).

   Turning off empty block creation is an optimization strategy to limit growth in chain size, although this only works if a majority of the nodes opt-in to this setting.
   ```shell
   cheqd-noded configure create-empty-blocks false
   ```
6. **Define the external peer-to-peer address**

   Unless you are running a node in a sentry/validator two-tier architecture, your node should be reachable on its peer-to-peer (P2P) port by other nodes. This can be defined by setting the `external-address` property which defines the externally reachable address. This can be defined using either the IP address or DNS name followed by the P2P port (Default: 26656).
   ```shell
   cheqd-noded configure p2p external-address <ip-address-or-dns-name:p2p-port>
   # Example
   # cheqd-noded configure p2p external-address 8.8.8.8:26656
   ```
   This is especially important if the node has no public IP address, e.g., if it's in a private subnet with traffic routed via a load balancer or proxy. Without the `external-address` property, the node will report a private IP address from its host network interface as its, which will be unreachable from the outside world. The node still works in this configuration, but only with limited unidirectional connectivity.
7. **Make the RPC endpoint available externally** (optional)

   This step is necessary only if you want to allow incoming client application connections to your node. Otherwise, the node will be accessible only locally. Further details about the RPC endpoints are available in the [cheqd node setup guide](https://github.com/cheqd/cheqd-node/blob/main/docs/setup-and-configure/README.md).
   ```shell
   cheqd-noded configure rpc-laddr "tcp://0.0.0.0:26657"
   ```
8. Download snapshot:

- ```shell
  wget http://cosmosia1.notional.ventures:11111/cheqd/data_20221211_052637.tar.gz 
  tar -xvzf data_20221211_052637.tar.gz
  mv data ~/.cheqdnode
  ```

## Setup Cosmovisor

Visit the Cosmovisor setup section to configure Cosmovisor for automated upgrades in the future:

[Setup Cosmovisor](<Setup Cosmovisor 261b9b86.md>)

## Start Cheqd

Finally, after setting the node up, we'll run Cheqd:

```shell
sudo systemctl start cosmovisor
```

## Post-installation checks

Once the `cheqd-noded` daemon is active and running, check that the node is connected to the cheqd testnet and catching up with the latest updates on the ledger.

### Checking node status via terminal

```shell
cheqd-noded status
```

In the output, look for the text `latest_block_height` and note the value. Execute the status command above a few times and make sure the value of `latest_block_height` has increased each time.

The node is fully caught up when the parameter `catching_up` returns the output `false`.

### Checking node status via the RPC endpoint

An alternative method to check a node's status is via the RPC interface if it has been configured.

- Remotely via the RPC interface: `cheqd-noded status --node <rpc-address>`
- By opening the JSONRPC over the HTTP status page through a web browser: `<node-address:rpc-port>/status`

## Next steps

At this stage, your node would be connected to the cheqd testnet as an observer node. Learn [how to configure your node as a validator node](https://github.com/cheqd/cheqd-node/blob/main/docs/validator-guide/README.md) to participate in staking rewards, block creation, and governance.
