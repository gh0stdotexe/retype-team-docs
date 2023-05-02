Adjusting our node pruning settings can make a big difference in the performance of our nodes.

## Pruning: custom

We can set up our nodes to a `custom` pruning config, using prime numbers for `pruning_keep_recent` and `pruning_interval`:

```shell
pruning="custom" && \
pruning_keep_recent="109" && \
pruning_keep_every="0" && \
pruning_interval="61" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.evmosd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.evmosd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.evmosd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.evmosd/config/app.toml
```

## Indexer: null

Setting our `indexer` to `null` is another simple, yet effective way to optimize our validator nodes:

```shell
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.source/config/config.toml
```

NOTE: do NOT use this on any node you wish to use as an RPC endpoint, as this will render the endpoint useless.
