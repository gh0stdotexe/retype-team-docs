# Disk Usage Optimization

Customize the configuration settings to lower the disk requirements for your validator node

Blockchain database tends to grow over time, depending e.g. on block speed and transaction amount. For Evmos, we are talking about close to 100GB of disk usage in first two weeks.

There are few configurations that can be done to reduce the required disk usage quite significantly. Some of these changes take full effect only when you do the configuration and start syncing from start with them in use.

## [#](https://evmos.dev/guides/validators/disk_optimization.html#indexing)Indexing

If you do not need to query transactions from the specific node, you can disable indexing. On `config.toml` set

Copyindexer = "null"&#x20;

If you do this on an already synced node, the collected index is not purged automatically, you need to delete it manually. The index is located under the database directory with the name `data/tx_index.db/`.

## [#](https://evmos.dev/guides/validators/disk_optimization.html#state-sync-snapshots)State-sync snapshots

I believe this was disabled by default on Evmos, but listing it in any case here. On `app.toml` set

```shell
snapshot-interval = 0 
```

Note that if state-sync was enabled on the network and working properly, it would allow one to sync a new node in a few minutes. But this node would not have the history.

## [#](https://evmos.dev/guides/validators/disk_optimization.html#configure-pruning)Configure pruning

By default, every 500th state, and the last 100 states are kept. This consumes a lot of disk space in the long run, and can be optimized with the following custom configuration:

```shell
pruning = "custom" 
pruning-keep-recent = "100" 
pruning-keep-every = "2" 
pruning-interval = "10"
```

Configuring `pruning-keep-recent = "0"` might sound tempting, but this will risk database corruption if the `evmosd` is killed for any reason. Thus, it is recommended to keep the few latest states.

## [#](https://evmos.dev/guides/validators/disk_optimization.html#logging)Logging

By default, the logging level is set to `info`, and this produces a lot of logs. This log level might be good when starting up to see that the node starts syncing properly. However, after you see the syncing is going smoothly, you can lower the log level to `warn` (or `error`). In `config.toml` set the following:

```shell
log_level = "warn" 
```

Also, ensure your log rotation is configured properly.

## [#](https://evmos.dev/guides/validators/disk_optimization.html#results)Results

Below is the disk usage after two weeks of Evmos Arsia Mons testnet. The default configuration results in disk usage of 90GB.

```shell
5.3G ./state.db 
70G ./application.db 
20K ./snapshots/metadata.db 
24K ./snapshots 
9.0G ./blockstore.db 
20K ./evidence.db 
1018M ./cs.wal 
4.7G ./tx_index.db 
90G .
```

This optimized configuration has reduced the disk usage to 17 GB.

```shell
17G .
1.1G ./cs.wal 
946M ./application.db 
20K ./evidence.db 
9.1G ./blockstore.db 
24K ./snapshots 
20K ./snapshots/metadata.db 
5.3G ./state.db
```

17G .1.1G ./cs.wal 946M ./application.db 20K ./evidence.db 9.1G ./blockstore.db 24K ./snapshots 20K ./snapshots/metadata.db 5.3G ./state.db
