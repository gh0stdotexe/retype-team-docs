After installing a new chain binary, sometimes we may need to reset the Cosmovisor symlink (for example, a chain restart after a halt, that's already had an upgrade before).

To do so:

```shell
# remove symlink & create new one
rm $DAEMON_HOME/cosmovisor/genesis/bin/junod && rm $DAEMON_HOME/cosmovisor/current
cp $HOME/go/bin/junod $DAEMON_HOME/cosmovisor/genesis/bin

# this should return <current version>
$DAEMON_HOME/cosmovisor/genesis/bin/junod version
```

<br>
