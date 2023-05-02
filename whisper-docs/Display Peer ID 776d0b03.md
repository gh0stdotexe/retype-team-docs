Our node peer IDs are a combination of our node key, IP address, and tendermint port `26656`.

This simple one-liner bash script will display our Peer ID:

```shell
echo "$(quasarnoded tendermint show-node-id)@$(curl ifconfig.me):18256"
```

<br>
