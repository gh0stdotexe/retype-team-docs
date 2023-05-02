When patching our `evmosd` binaries, we'll need to use a special `Tendermint-MEV` version (released by the [==Skip.money== ](https://skip.money/validators "Skip")team) for our validator in order to continue processing bundles on the network post-upgrade.

This is easily done by cloning the `github.com/skip-mev/evmos` repository and building the binary from there:

```shell
git clone https://github.com/skip-mev/evmos evmos-mev
cd evmos-mev
git checkout v11.0.2-mev
make install
```

Verify the version output matches the expected version:

```shell
evmosd version
# should output:
# v11.0.2-mev
```

We'll copy over our binary into Cosmovisor, as usual:

```shell
sudo systemctl stop cosmovisor
cp $GOPATH/bin/evmosd ~/.evmosd/cosmovisor/upgrades/v11.0.0/bin/evmosd
~/.evmosd/cosmovisor/upgrades/v11.0.0/bin/evmosd version
# should output v11.0.2-mev
```

Restart Cosmovisor & confirm Skip is working via the [Skip.money](https://skip.money/validators "Skip") dashboard.

```shell
sudo systemctl restart cosmovisor && journalctl -fu cosmovisor
```

<br>
