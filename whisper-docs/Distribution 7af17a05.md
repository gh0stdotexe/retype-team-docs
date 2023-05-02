The distribution module allows you to manage your staking rewards

# Available Commands

| Name                          | Description                                                                                                                                            |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| commission                    | Query distribution validator commission                                                                                                                |
| community-pool                | Query the amount of coins in the community pool                                                                                                        |
| params                        | Query distribution params                                                                                                                              |
| rewards                       | Query all distribution delegator rewards or rewards from a particular validator                                                                        |
| slashes                       | Query distribution validator slashes.                                                                                                                  |
| validator-outstanding-rewards | Query distribution outstanding (un-withdrawn) rewards for a validator and all their delegations                                                        |
| fund-community-pool           | Funds the community pool with the specified amount                                                                                                     |
| set-withdraw-addr             | Set the withdraw address for rewards associated with a delegator address                                                                               |
| withdraw-all-rewards          | Withdraw all rewards for a single delegator                                                                                                            |
| withdraw-rewards              | Withdraw rewards from a given delegation address, and optionally withdraw validator commission if the delegation address given is a validator operator |

## osmosisd query distribution commission

Query validator commission rewards from delegators to that validator.

```
osmosisd query distribution commission [validator] [flags]
```

## osmosisd query distribution community-pool

Query all coins in the community pool which is under Governance control.

```
osmosisd query distribution community-pool [flags]
```

## osmosisd query distribution params

Query distribution params.

```
osmosisd query distribution params [flags]
```

## osmosisd query distribution rewards

Query all rewards earned by a delegator, optionally restrict to rewards from a single validator.\\

```
osmosisd query distribution rewards [delegator-addr] [validator-addr] [flags]
```

## osmosisd query distribution slashes

Query all slashes of a validator for a given block range.

```
osmosisd query distribution slashes [validator] [start-height] [end-height] [flags]
```

## osmosisd query distribution validator-outstanding-rewards

Query distribution outstanding (un-withdrawn) rewards for a validator and all their delegations.

```
osmosisd query distribution validator-outstanding-rewards [validator] [flags]
```

## osmosisd tx distribution fund-community-pool

Funds the community pool with the specified amount.

```
osmosisd tx distribution fund-community-pool [amount] [flags]
```

## osmosisd tx distribution set-withdraw-addr

Set the withdraw address for rewards associated with a delegator address.

```
osmosisd tx distribution set-withdraw-addr [withdraw-addr] [flags]
```

## osmosisd tx distribution withdraw-all-rewards

Withdraw all rewards for a single delegator.

```
osmosisd tx distribution withdraw-all-rewards [flags]
```

## osmosisd tx distribution withdraw-rewards

Withdraw rewards from a given delegation address, and optionally withdraw validator commission if the delegation address given is a validator operator.

```
osmosisd tx distribution withdraw-rewards [validator-addr] [flags]
```

<br>
