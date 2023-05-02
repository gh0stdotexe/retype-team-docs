Use this guide to upgrade our Axelar testnet nodes (both validator + any backup nodes) – simply replace the old `axelard` + `vald` version with the latest version when upgrading.

Be sure to check log files if any failures occur when running the upgrade scripts.

---

### Kill Node processes:

```shell
pkill -f 'axelard start'
# Validators need to also stop vald/tofnd
pkill -f vald
pkill -f tofnd
ulimit -n 16384

#### OLD KILL COMMANDS - DEPRECATED

kill -9 $(pgrep -f "axelard")
# Validators need to also stop vald/tofnd
kill -9 $(pgrep -f "axelard vald-start")
kill -9 $(pgrep tofnd)
```

### OPTIONAL: Backup chain state in case of failure

```shell
# testnet
cp -r ~/.axelar_testnet/.core/data ~/.axelar-lisbon-3-upgrade-0.19/.core/data
```

### Pull latest Repo:

```shell
cd axelarate-community
git status
git branch
git pull origin main
```

### OPTIONAL: Add Mnemonic files to \~/backup:

```shell
nano validatormnemonic.txt
nano broadcastermnemonic.txt
nano tofndmnemonic.txt 
nano priv_validator_key.json
```

### TESTNET: Run Validator Script:

```shell
./scripts/node.sh -n testnet -a v0.32.0
```

### &#x20;TESTNET: Run VALD/TOFND Script:

```shell
./scripts/validator-tools-host.sh -n testnet -a v0.32.0 -q v0.10.1
```

### Check Config.toml (SPECIFY WHICH ONE, THOUGH):

```shell
[[axelar_bridge_evm]]
name = "Arbitrum"
rpc_addr = "https://arbitrum-goerli.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Aurora"
rpc_addr = "https://aurora-testnet.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Avalanche"
rpc_addr = "https://avalanche-fuji.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Base"
rpc_addr = "https://cool-purple-diamond.base-goerli.quiknode.pro/f5ab69c8bf727274e90b473379ab32fbe8da6d4d"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Binance"
rpc_addr = "https://frequent-damp-snow.bsc-testnet.quiknode.pro/564de9843e88505cdc37dc9e87cace0faab94f06/"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Celo"
rpc_addr = "https://celo-alfajores.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Ethereum-2"
rpc_addr = "https://goerli.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Fantom"
rpc_addr = "https://nd-217-244-041.p2pify.com/fdb7ac12df66ec9c014a6a1620cc955d"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Kava"
rpc_addr = "http://51.81.66.123:8545"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Moonbeam"
rpc_addr = "https://moonbeam-alpha.api.onfinality.io/rpc?apikey=be6531e0-5a7d-487d-962c-16c2c7f733e5"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Optimism"
rpc_addr = "https://optimism-goerli.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

[[axelar_bridge_evm]]
name = "Polygon"
rpc_addr = "https://polygon-mumbai.infura.io/v3/1de6e369dbfa41b084b699b9305b424c"
start-with-bridge = true

##### message broadcasting options #####
[broadcast]

broadcaster-account = "broadcaster"
gas-adjustment = 1.0
max-retries = 10
min-timeout = "4s"
```

### Run Health Check:

```shell
axelard health-check --tofnd-host localhost --operator-addr $(cat ~/.axelar_testnet/validator.bech) --node http://localhost:26657/
```

---

## Troubleshooting & MISC:

General commands and tips to fix various items (like registering chain maintainers that have deregistered, etc.).

### Register Chain Maintainer:

```shell
axelard tx nexus register-chain-maintainer base --from broadcaster --node http://65.21.226.198:26657 --chain-id axelar-testnet-lisbon-3 --gas auto --gas-adjustment 1.2
```

<br>
