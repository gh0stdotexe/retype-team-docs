Original Documentation:

[Avalanche | Axelar Network](https://docs.axelar.dev/validator/external-chains/avalanche "Avalanche | Axelar Network")

GitHub Repo:

[GitHub - ava-labs/avalanche-docs](https://github.com/ava-labs/avalanche-docs "GitHub - ava-labs/avalanche-docs")

Testnet Explorer:

[SnowTrace: Avalanche Testnet C-Chain Blockchain Explorer](https://testnet.snowtrace.io/ "SnowTrace: Avalanche Testnet C-Chain Blockchain Explorer")

---

## Install AvalancheGo

In this guide, we will be using a bash script created by the Avalanche team, which will automatically set up a running node with minimum user input required.

### 1. Download avalanchego-installer

```shell
wget -nd -m https://raw.githubusercontent.com/ava-labs/avalanche-docs/master/scripts/avalanchego-installer.sh
```

### 2. Set permission and run the script

```shell
chmod 755 avalanchego-installer.sh
./avalanchego-installer.sh
```

⚠️

By default, the network will start synchronizing on the Mainnet network. If you are trying to set up a node on the testnet network (Avalanche Fuji), you need to stop the `avalanchego.service`, edit the `node.json` configuration file and restart the service.

```shell
sudo systemctl stop avalanchego
nano  /home/whispernode/.avalanchego/configs/node.json
```

Change `"network-id"` to `"fuji"`, save the file. Then enable [state sync](https://docs.avax.network/nodes/maintain/chain-config-flags#c-chain-configs "Chain Configs | Avalanche Docs"):

```shell
# If it does not exist, create the following directory and file:

$HOME/.avalanchego/configs/chains/C/config.json

#Add the line:
{
"state-sync-enabled": true
}

# Delete old DB (if it exists)
## (Be careful not to delete the "staking" directory!)
$HOME/.avalanchego/db
```

```shell
sudo systemctl start avalanchego
```

The script will start and prompt you for information about your server environment. Follow the required steps, enter your network information and confirm the installation. The script will then create and start the `avalanchego.service` for you automatically. To check if the service is running or follow the logs, use the following commands:

```shell
sudo systemctl status avalanchego
sudo journalctl -u avalanchego -f
```

Now you should be synchronizing on the Avalanche Testnet network. Once the network is synced, you will see a message similar to:

`health/service.go#130: "isBootstrapped" became healthy with: {"timestamp":"2021-12-27T21:35:36.879654389+01:00","duration":6943,"contiguousFailures":0,"timeOfFirstFailure":null}`

## Test your Avalanche RPC connection

Once your node is fully synced, you can run a cURL request to see the status of your node:

```shell
curl -X POST --data '{"jsonrpc": "2.0","method": "info.isBootstrapped","params":{"chain":"C"},"id":1}' -H 'content-type:application/json;' localhost:9650/ext/info
```

If the node is successfully synced, the output from above will print `{"jsonrpc":"2.0","result":{"isBootstrapped":true},"id":1}`

### EVM RPC endpoint URL

Axelar Network will be connecting to the C-Chain, which is the EVM compatible blockchain, so your `rpc_addr` should be exposed in this format:

```shell
http://65.108.108.169:9650/ext/bc/C/rpc
```

---

## Updating Avalanche

[Upgrade Your AvalancheGo Node | Avalanche Docs](https://docs.avax.network/nodes/maintain/upgrade-your-avalanchego-node "Upgrade Your AvalancheGo Node | Avalanche Docs")

## Node installed using the installer script[​](https://docs.avax.network/nodes/maintain/upgrade-your-avalanchego-node#node-installed-using-the-installer-script "Direct link to heading")

If you installed your node using the [installer script](https://docs.avax.network/nodes/build/set-up-node-with-installer), to upgrade your node, just run the installer script again.

```shell
./avalanchego-installer.sh --version v1.9.0-fuji-startup
```

It will detect that you already have AvalancheGo installed:

```shell
AvalancheGo installer
---------------------
Preparing environment...
Found 64bit Intel/AMD architecture...
Found AvalancheGo systemd service already installed, switching to upgrade mode.
Stopping service...
```

It will then upgrade your node to the latest version, and after it's done, start the node back up, and print out the information about the latest version:

```shell
Node upgraded, starting service...
New node version:
avalanche/1.1.1 [network=mainnet, database=v1.0.0, commit=f76f1fd5f99736cf468413bbac158d6626f712d2]
Done!
```

And that is it, your node is upgraded to the latest version.

If you installed your node manually, proceed with the rest of the tutorial.
