Kill Node processes:

```shell
kill -9 $(pgrep -f "axelard")
# Validators need to also stop vald/tofnd
kill -9 $(pgrep -f "axelard vald-start")
kill -9 $(pgrep tofnd)



pkill -f 'axelard start'
# Validators need to also stop vald/tofnd
pkill -f vald
pkill -f tofnd
```

Backup .Axelar in case of failure:

```shell
cp -r ~/.axelar/.core/data ~/.axelar-dojo-1-upgrade-0.26/.core/data
```

\
Pull latest Repo:

```shell
## upgrade same branch 

git checkout main
git pull

## For separate branch)

git branch
git status
git commit -a
git checkout main
git pull origin main
git checkout whispernode
git rebase main
git status
```

Edit the configuration settings in `~/axelarate-community/configuration/config.toml`to add the external address and RPC nodes:

```shell
nano ~/axelarate-community/configuration/config.toml
```

```shell
....

# Address to advertise to peers for them to dial
# If empty, will use the same port as the laddr,
# and will introspect on the listener or use UPnP
# to figure out the address. ip and port are required
# example: 159.89.10.97:26656
# external_address = ""
external_address = "23.88.3.237:26656"
# Comma separated list of seed nodes to connect to
seeds = ""

.....


[[axelar_bridge_evm]]
name = "Ethereum"
rpc_addr = "https://indulgent-smart-asphalt.quiknode.pro/52fdde92abaca124ba8beb5c3871ac74f086a11c/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Avalanche"
rpc_addr = "https://cosmological-long-sky.avalanche-mainnet.quiknode.pro/577d89d3383f5bfc6f315d896d81c943c5a34f71/ext/bc/C/rpc"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Fantom"
rpc_addr = "https://sparkling-alpha-hill.fantom.quiknode.pro/88c93caeb4b82fcd0a9f96817f51dd895c88f7ca/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Moonbeam"
rpc_addr = "https://moonbeam.api.onfinality.io/rpc?apikey=3860587a-2761-4779-97f4-91f3501336a6"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Polygon"
rpc_addr = "https://polygon-mainnet.g.alchemy.com/v2/dMTAZm0aE4pQ0Ke-7PzJlfm9WXhs3Ja4"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Binance"
rpc_addr = "https://polished-still-shape.bsc.quiknode.pro/d257f6670add08fce9570108dd18d9ac462b545c/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Aurora"
rpc_addr = "https://aurora-mainnet.infura.io/v3/68b35cfb849f45dd9b86bb6b46b5ee3c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Celo"
rpc_addr = "https://frequent-divine-thunder.celo-mainnet.quiknode.pro/e8a6f0cf18761363af49f84a6be3a9c37b4456ec/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Arbitrum"
rpc_addr = "https://arb-mainnet.g.alchemy.com/v2/jGlE2voMlLO9EKcx5M5-1kHxRvFwieIG"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Kava"
rpc_addr = "http://65.108.138.80:39545"
start-with-bridge = true
```

Run the Launch Validator Node.sh Script being sure to check relevant flags

|   | -h, --help Print this help and exit                                             |
| - | ------------------------------------------------------------------------------- |
|   | -v, --verbose Print script debug info                                           |
|   | -r, --reset-chain Reset all chain data (erases current state including secrets) |
|   | -a, --axelar-core-version Version of axelar core to checkout                    |
|   | -d, --root-directory Directory for data. \[default: $HOME/.axelar\_testnet]     |
|   | -n, --network Core Network to connect to \[devnet\|testnet\|mainnet]            |
|   | -t, --tendermint-key-path Path to tendermint key                                |
|   | -t, --tendermint-key-path Path to tendermint key                                |
|   | -t, --tendermint-key-path Path to tendermint key                                |
|   | -c, --chain-id Axelard Chain ID \[default: axelar-testnet-lisbon]               |
|   | -k, --node-moniker Node Moniker \[default: hostname]                            |

Likely example script format Run:

```shell
./scripts/node.sh -n mainnet -a v0.31.2
```

Check Validator Logs to make sure the validator is producing blocks:

```shell
tail -f /home/whispernode/.axelar/logs/axelard.log
```

Once successfully CAUGHT UP on blocks Run VALD/TOFND Script being sure to check relevant flags as listed above:

```shell
./scripts/validator-tools-host.sh -n mainnet -a v0.31.2 -q v0.10.1
```

Check TOFND and VALD Logs:

```shell
tail -f /home/whispernode/.axelar/logs/tofnd.log
tail -f /home/whispernode/.axelar/logs/vald.log
```

Check configuration settings in \~/.axelar/.core/config/config.toml and \~/.axelar/.vald/config/config.toml to ensure RPC endpoints are input correctly after running the scripts (It's OK if the .core config file does not have these):

```shell
nano ~/.axelar/.core/config/config.toml
nano ~/.axelar/.vald/config/config.toml
```

```shell
# Bridges, with Polkachu / Endpoint Provider RPCs

[[axelar_bridge_evm]]
name = "Ethereum"
rpc_addr = "https://indulgent-smart-asphalt.quiknode.pro/52fdde92abaca124ba8beb5c3871ac74f086a11c/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Avalanche"
rpc_addr = "https://cosmological-long-sky.avalanche-mainnet.quiknode.pro/577d89d3383f5bfc6f315d896d81c943c5a34f71/ext/bc/C/rpc"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Fantom"
rpc_addr = "https://sparkling-alpha-hill.fantom.quiknode.pro/88c93caeb4b82fcd0a9f96817f51dd895c88f7ca/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Moonbeam"
rpc_addr = "https://moonbeam.api.onfinality.io/rpc?apikey=3860587a-2761-4779-97f4-91f3501336a6"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Polygon"
rpc_addr = "https://polygon-mainnet.g.alchemy.com/v2/dMTAZm0aE4pQ0Ke-7PzJlfm9WXhs3Ja4"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Binance"
rpc_addr = "https://polished-still-shape.bsc.quiknode.pro/d257f6670add08fce9570108dd18d9ac462b545c/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Aurora"
rpc_addr = "https://aurora-mainnet.infura.io/v3/68b35cfb849f45dd9b86bb6b46b5ee3c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Celo"
rpc_addr = "https://frequent-divine-thunder.celo-mainnet.quiknode.pro/e8a6f0cf18761363af49f84a6be3a9c37b4456ec/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Arbitrum"
rpc_addr = "https://arb-mainnet.g.alchemy.com/v2/jGlE2voMlLO9EKcx5M5-1kHxRvFwieIG"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Kava"
rpc_addr = "http://65.108.138.80:39545"
start-with-bridge = true
```

Run Health Check to ensure all 3 processes are "passed":

```shell
axelard health-check --tofnd-host localhost --operator-addr $(cat ~/.axelar/validator.bech) --node http://localhost:26657/
```

<br>
