Get standard debug info from the `osmosisd` daemon:

```shell
osmosisd status
```

Check if your node is catching up:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

Get your node ID:

```shell
osmosisd tendermint show-node-id
```

> Your peer address will be the result of this plus host and port, i.e. `<id>@<host>:26656` if you are using the default port.

Set the default chain for commands to use:

```shell
osmosisd config chain-id osmosis-1
```

Get your `valoper` address:

```shell
osmosisd keys show <your-key-name> -a --bech val
```

See keys on the current box:

```shell
osmosisd keys list
```

Import a key from a mnemonic:

```shell
osmosisd keys add <new-key-name> --recover
```

Export a private key (warning: don't do this unless you know what you're doing!)

```shell
osmosisd keys export <your-key-name> --unsafe --unarmored-hex
```

Withdraw rewards (including validator commission), where `osmovaloper1...` is the validator address:

```shell
osmosisd tx distribution withdraw-rewards <osmovaloper1...> --from <your-key>  --commission
```

```
osmosisd tx distribution withdraw-rewards $(osmosisd keys show --bech=val -a whispernode) --from whispernode --commission --gas-prices 0.15uosmo --chain-id osmosis-1 
```

Send Transaction:

```
osmosisd tx bank send whispernode osmo1p7s8vyxcdvhrenzht54zwgg7uxdvts6nv43ksc 31000000uosmo --gas-prices 0.25uosmo --chain-id osmosis-1
```

Stake:

```shell
osmosisd tx staking delegate <osmovaloper1...> <AMOUNT>uosmo --from <your-key>
```

Find out what the JSON for a command would be using `--generate-only`:

```shell
osmosisd tx bank send $(osmosisd keys show <your-key-name> -a) <recipient addr> <AMOUNT>uosmo --generate-only
```

<br>
