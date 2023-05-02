The slashing module can unjail your validator

# Available Commands

| Name   | Description                                                     |
| ------ | --------------------------------------------------------------- |
| unjail | Unjail validator previously jailed for downtime or double-sign. |

## osmosisd tx slashing unjail

Unjail validator previously jailed for downtime.

```
osmosisd tx slashing unjail [flags]
```

## Unjail a validator

The following example will unjail a validator using its validator operator (owner) key:

```
osmosisd tx slashing unjail --from osmo1ludczrvlw36fkur9vy49lx4vjqhppn30h42ufg --chain-id osmosis-1
```

> **TIP**
>
> The validator operator key must be stored in the local keystore.
