### Make a Text Proposal

```shell
chihuahuad tx gov submit-proposal \
  --title Proposal: Let's Get People Their Airdrop! \
  --description This is a proposal. Vote Yes if you like LUM. \
  --type Text \
  --deposit 100000000ulum \
  --from Exidio
```

### Withdraw Rewards + Commissions

```shell
chihuahuad tx bank distribution withdraw-rewards $(lumd keys show --bech=val -a Exidio) --commission --gas-prices 0.25ulum --chain-id lum-network-1 --from Exidio
```

### Governance Deposit

```shell
chihuahuad tx gov deposit 4 "500000000ulum" --gas-prices 0.25ulum --chain-id lum-network-1 --from Exidio
```

Get standard debug info from the `chihuahuad` daemon:

```shell
chihuahuad status
```

### Check if your node is catching up:

```shell
# Query via the RPC (default port: 26657)
curl -fsLS http://127.0.0.1:26657/status | jq -r '.result.sync_info.latest_block_height'
```

### Get your node ID:

```shell
chihuahuad tendermint show-node-id
```

> Your peer address will be the result of this plus host and port, i.e. `<id>@<host>:26656` if you are using the default port.

Set the default chain for commands to use:

```shell
chihuahuad config chain-id chihuahua-1
```

Get your `valoper` address:

```shell
chihuahuad keys show <your-key-name> -a --bech val
```

See keys on the current box:

```shell
chihuahuad keys list
```

Import a key from a mnemonic:

```shell
chihuahuad keys add <new-key-name> --recover
```

Export a private key (warning: don't do this unless you know what you're doing!)

```shell
chihuahuad keys export <your-key-name> --unsafe --unarmored-hex
```

Withdraw rewards (including validator commission), where `osmovaloper1...` is the validator address:

```shell
chihuahuad tx distribution withdraw-rewards <osmovaloper1...> --from <your-key>  --commission
```

Stake:

```shell
chihuahuad tx staking delegate <osmovaloper1...> <AMOUNT>uhuahua --from <your-key>
```

Find out what the JSON for a command would be using `--generate-only`:

```shell
chihuahuad tx bank send $(chihuahuad keys show <your-key-name> -a) <recipient addr> <AMOUNT>uhuahua --generate-only
```

<br>
