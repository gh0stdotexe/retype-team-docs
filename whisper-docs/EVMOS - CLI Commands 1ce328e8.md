Get standard debug info from the `comdex` daemon:

```shell
comdex status
```

Check if your node is catching up:

```shell
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

Get your node ID:

```shell
comdex tendermint show-node-id
```

> Your peer address will be the result of this plus host and port, i.e. `<id>@<host>:26656` if you are using the default port.

Set the default chain for commands to use:

```shell
comdex config chain-id comdex-1
```

Get your `valoper` address:

```shell
comdex keys show <your-key-name> -a --bech val
```

See keys on the current box:

```shell
comdex keys list
```

Import a key from a mnemonic:

```shell
comdex keys add <new-key-name> --recover
```

Export a private key (warning: don't do this unless you know what you're doing!)

```shell
comdex keys export <your-key-name> --unsafe --unarmored-hex
```

Withdraw rewards (including validator commission), where `comdexvaloper1...` is the validator address:

```shell
comdex tx distribution withdraw-rewards <comdexvaloper1...> --from <your-key>  --commission
```

Stake:

```shell
comdex tx staking delegate <comdexvaloper1...> <AMOUNT>ucmdx --from <your-key>
```

Find out what the JSON for a command would be using `--generate-only`:

```shell
comdex tx bank send $(comdex keys show <your-key-name> -a) <recipient addr> <AMOUNT>ucmdx --generate-only
```

<br>
