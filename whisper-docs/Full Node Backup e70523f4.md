# Backup a Full Node

Why you might want to backup your node ID:

- In case you are a seed node.
- In case you are a persistent peer for other full nodes, archive nodes, or validators.
- In case you manage a setup of senty nodes and use node IDs in your config files.

To see the associated public key:

```shell
<networkd> tendermint show-validator
```

# Full Node Private key

- Backup `~/.secretd/config/node_key.json`.

# Full Node Data

- Gracefully shut down the node:

```shell
sudo systemctl stop secret-node
```

1. Backup the `~/.<networkd>/data/` directory except for the \
   `~/.<networkd>/data/priv_validator_state.json` file.
2. Backup the `~/.<networkd>/.compute/` directory.
3. Restart the node:

```shell
sudo systemctl start secret-node
```

<br>
