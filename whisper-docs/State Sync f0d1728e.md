### Optional: Reset Validator State

```shell
nolusd tendermint unsafe-reset-all --home ~/.nolus
```

### Download Nolus wasm folder

Replace existing `wasm` folder (if one exists):

```shell
curl -L https://share2.utsa.tech/nolus/wasm-nolus.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus --strip-components 2
```

### Set Statesync Peers:

```shell
peers="12b146cd82c7142e9d8aeb4f246499927ecb1c0f@217.13.223.167:36656"
sed -i.bak -e "s/^persistent_peers =./persistent_peers = "$peers"/" $HOME/.nolus/config/config.toml
```

### Setup Statesync RPC + Get Trusted Height:

```shell
SNAP_RPC=https://t-nolus.rpc.utsa.tech:443

# grab trusted height
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```

### Run Statesync / Add Height to config.toml

```shell
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).$|\1true|
s|^(rpc_servers[[:space:]]+=[[:space:]]+).$|\1"$SNAP_RPC,$SNAP_RPC"|
s|^(trust_height[[:space:]]+=[[:space:]]+).$|\1$BLOCK_HEIGHT|
s|^(trust_hash[[:space:]]+=[[:space:]]+).$|\1"$TRUST_HASH"|
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1""|" $HOME/.nolus/config/config.toml
```

### Restart Daemon

```shell
sudo systemctl restart cosmovisor-13 && journalctl -fu cosmovisor-13
```

<br>
