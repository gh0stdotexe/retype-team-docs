### Archive all blockchain data.

An archive node keeps all the past blocks. An archive node makes it convenient to query the past state of the chain at any point in time. Finding out what an account's balance, stake size, etc at a certain block was, or which extrinsics resulted in a certain state change are fast operations when using an archive node. However, an archive node takes up a lot of disk space - nearly 400GB for secret-2 as of May 31, 2021.

To setup your archive node you can follow the instructions below:

First follow the [**Full Node Guide**](https://app.nuclino.com/WhisperNode/Validator-Nodes/Full-Node-Prep-7eeb60f4-d35b-48af-8d32-f886f52e52c3 "Nuclino")**.**

Now stop the full node you want to convert to an archive node.

```
sudo systemctl stop secret-node
```

Full nodes should edit their `.secretd/config/config.toml`:

```
nano .secretd/config/app.toml
```

Proceed to make the following changes:

```shell
# Pruning sets the pruning strategy: syncable, nothing, everything
# syncable: only those states not needed for state syncing will be deleted (keeps last 100 + every 10000th)
# nothing: all historic states will be saved, nothing will be deleted (i.e. archiving node)
# everything: all saved states will be deleted, storing only the current state
pruning = "nothing"
```

Now you must reset the data on your full node, so it can sync in archive mode.

```
secretd unsafe-reset-all
```

Now proceed to restart your secret node with the following command.

```
sudo systemctl start secret-node
```

You now have an Archive node running!
