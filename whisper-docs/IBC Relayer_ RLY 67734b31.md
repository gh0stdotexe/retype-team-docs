**Original Guide:**

[GitHub - cosmos/relayer: An IBC relayer for ibc-go](https://github.com/cosmos/relayer "GitHub - cosmos/relayer: An IBC relayer for ibc-go")

---

# Introduction to RLY

In IBC, blockchains do not directly pass messages to each other over the network. This is where `relayer` comes in. A relayer process monitors for updates on opened paths between sets of [IBC](https://ibcprotocol.org/) enabled chains. The relayer submits these updates through specific message types to the counterparty chain. Clients are then used to track and verify the consensus state.

In addition to relaying packets, this relayer can open paths across chains, thus creating clients, connections, and channels.

Additional information on how IBC works can be found [here](https://ibc.cosmos.network/).

| **Relayer** | **IBC-Go**           |
| ----------- | -------------------- |
| v1.0.0      | ibc-go v1, ibc-go v2 |
| v2.0.0      | ibc-go v3            |

\*\*If you are updating the relayer from any version prior to `v2.0.0-rc1`, your current config file is not compatible. See: [config\_migration](https://github.com/cosmos/relayer/blob/main/docs/config_migration.md)

---

## Table Of Contents

- [Basic Usage - Relaying Across Chains](https://github.com/cosmos/relayer#Basic-Usage---Relaying-Packets-Across-Chains)
- [Create Path Across Chains](https://github.com/cosmos/relayer/blob/main/docs/create-path-across-chain.md)
- [Troubleshooting](https://github.com/cosmos/relayer/blob/main/docs/troubleshooting.md)
- [Features](https://github.com/cosmos/relayer/blob/main/docs/features.md)
- [Relayer Terminology](https://github.com/cosmos/relayer/blob/main/docs/terminology.md)
- [New Chain Implementation](https://github.com/cosmos/relayer/blob/main/docs/chain_implementation.md)
- [Recommended Pruning Settings](https://github.com/cosmos/relayer/blob/main/docs/node_pruning.md)
- [Demo](https://github.com/cosmos/relayer/blob/main/docs/demo.md)

---

## Basic Usage - Relaying Packets Across Chains

> The `-h` (help) flag tailing any `rly` command will be your best friend. USE THIS IN YOUR RELAYING JOURNEY.

---

1. **Clone, checkout, and install the latest release (**[**releases page**](https://github.com/cosmos/relayer/releases)**).**

   [*Go*](https://go.dev/doc/install) *needs to be installed and a proper Go environment needs to be configured*
   ```shell
   git clone https://github.com/cosmos/relayer.git
   cd relayer && git checkout v2.0.0-rc3
   make install
   ```
2. **Initialize the relayer's configuration directory/file.**
   ```shell
   rly config init
   ```
   **Default config file location:** `~/.relayer/config/config.yaml`

   By default, transactions will be relayed with a memo of `rly(VERSION)` e.g. `rly(v2.0.0)`.

   To customize the memo for all relaying, use the `--memo` flag when initializing the configuration.
   ```shell
   rly config init --memo "My custom memo"
   ```
   Custom memos will have `rly(VERSION)` appended. For example, a memo of `My custom memo` running on relayer version `v2.0.0` would result in a transaction memo of `My custom memo | rly(v2.0.0)`.

   The `--memo` flag is also available for other `rly` commands also that involve sending transactions such as `rly tx link` and `rly start`. It can be passed there to override the `config.yaml` value if desired.

   To omit the memo entirely, including the default value of `rly(VERSION)`, use `-` for the memo.
3. **Configure the chains you want to relay between.**

   In our example, we will configure the relayer to operate on the canonical path between the Cosmos Hub and Osmosis.\
   \
   The `rly chains add` command fetches chain meta-data from the [chain-registry](https://github.com/cosmos/chain-registry) and adds it to your config file.
   ```shell
   rly chains add cosmoshub osmosis
   ```
   Adding chains from the chain-registry randomly selects an RPC address from the registry entry.\
   If you are running your own node, manually go into the config and adjust the `rpc-addr` setting.
   > NOTE: `rly chains add` will check the liveliness of the available RPC endpoints for that chain in the chain-registry.\
   > It is possible that the command will fail if none of these RPC endpoints are available. In this case, you will want to manually add the chain config.
   To add the chain config files manually, example config files have been included [here](https://github.com/cosmos/relayer/tree/main/docs/example-configs/)
   ```shell
   rly chains add --url https://raw.githubusercontent.com/cosmos/relayer/main/docs/example-configs/cosmoshub-4.json cosmoshub
   rly chains add --url https://raw.githubusercontent.com/cosmos/relayer/main/docs/example-configs/osmosis-1.json osmosis
   ```
4. **Import OR create new keys for the relayer to use when signing and relaying transactions.**
   > `key-name` is an identifier of your choosing.
   If you need to generate a new private key you can use the `add` subcommand.
   ```shell
   rly keys add cosmoshub [key-name]  
   rly keys add osmosis [key-name]  
   ```
   If you already have a private key and want to restore it from your mnemonic you can use the `restore` subcommand.
   ```shell
   rly keys restore cosmoshub [key-name] "mnemonic words here"
   rly keys restore osmosis [key-name] "mnemonic words here"
   ```
5. **Edit the relayer's** **`key`** **values in the config file to match the** **`key-name`'s chosen above.**
   > This step is necessary if you chose a `key-name` other than "default"
   Example:
   ```shell
   - type: cosmos
      value:
      key: YOUR-KEY-NAME-HERE
      chain-id: cosmoshub-4
      rpc-addr: http://localhost:26657
   ```
6. **Ensure the keys associated with the configured chains are funded.**
   > Your configured addresses will need to contain some of the respective native tokens for paying relayer fees.
   \
   You can query the balance of each configured key by running:
   ```shell
   rly q balance cosmoshub WhisperNode
   rly q balance osmosis WhisperNode
   ```
7. **Configure path meta-data in the config file.**\
   We have the chain meta-data configured, now we need path meta-data. For more info on `path` terminology visit [here](https://github.com/cosmos/relayer/blob/main/docs/troubleshooting.md).
   > NOTE: Thinking of chains in the config as "source" and "destination" can be confusing. Be aware that most path are bi-directional.
   \


   `rly paths fetch` will check for IBC path metadata from the [chain-registry](https://github.com/cosmos/chain-registry/tree/master/_IBC) and add these paths to your config file.
   ```shell
   rly paths fetch
   ```
   > **NOTE:** Don't see the path metadata for paths you want to relay on?\
   > Please open a PR to add this metadata to the GitHub repo!
8. **Configure the channel filter.**

   By default, the relayer will relay packets over all channels on a given connection.\
   \
   Each path has a `src-channel-filter` which you can utilize to specify which channels you would like to relay on.\
   \
   The `rule` can be one of three values:
   - `allowlist` which tells the relayer to relay on *ONLY* the channels in `channel-list`
   - `denylist` which tells the relayer to relay on all channels *BESIDES* the channels in `channel-list`
   - empty value, which is the default setting, and tells the relayer to relay on all channels
   \


   Since we are only worried about the canonical channel between the Cosmos Hub and Osmosis our filter settings would look like the following.\
   \
   Example:
   ```shell
   hubosmo:
      src:
          chain-id: cosmoshub-4
          client-id: 07-tendermint-259
          connection-id: connection-257
      dst:
          chain-id: osmosis-1
          client-id: 07-tendermint-1
          connection-id: connection-1
      src-channel-filter:
              rule: allowlist
              channel-list: [channel-141]  
   ```
   > Because two channels between chains are tightly coupled, there is no need to specify the dst channels. If you only know the "dst" channel-ID you can query the "src" channel-ID by running: `rly q channel <dst_chain_name> <dst_channel_id> <port> | jq '.channel.counterparty.channel_id'`
9. **Finally, we start the relayer on the desired path.**

   The relayer will periodically update the clients and listen for IBC messages to relay.
   ```shell
   rly paths list
   rly start [path]
   # Optionally you can omit the `path` argument to start all configured paths
   rly start 
   ```
   You will need to start a separate shell instance for each path you wish to relay over.
   > When running multiple instances of `rly start`, you will need to use the `--debug-addr` flag and provide an address:port. You can also pass an empty string `''` to turn off this feature or pass `localhost:0` to randomly select a port.
   \[[TROUBLESHOOTING](https://github.com/cosmos/relayer/blob/main/docs/troubleshooting.md)]

---

## Security Notice

If you would like to report a security-critical bug related to the relayer repo, please reach out to @jackzampolin or @Ethereal0ne on telegram.

## Code of Conduct

The Cosmos community is dedicated to providing an inclusive and harassment-free experience for contributors. Please visit the [Code of Conduct](https://github.com/cosmos/relayer/blob/main/CODE_OF_CONDUCT.md) for more information.

---

[Create Path Across Chains -->](https://github.com/cosmos/relayer/blob/main/docs/create-path-across-chain.md)
