---

We can use the `rollback` command to mitigate issues caused by app hash errors caused by state non-determinism. This is a new command in `Tendermint`.

You need to first stop the `seid` process, then run:

```shell
seid rollback
```

to revert the tendermint state to the previous height;

OR, if you run&#x20;

```shell
seid rollback --hard
```

it will roll back `Tendermint` and the current app state.

## FAQ:

Generally, we should first create an IAVL dump of the bad/corrupted block height using:

```shell
seid debug iavl-dump <height>
```

so that the team can investigate the source of the nondeterminism issue.
