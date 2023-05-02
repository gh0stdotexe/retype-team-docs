Link to original docs:

[slashing - Junø](https://docs.junonetwork.io/cli/modules/slashing "slashing - Junø")

---

## Unjail a Validator

Downtime slashing requires that a validator node be "unjailed" in order to join the active set again. Accomplishing this is extremely simple:

```shell
comdex tx slashing unjail --from WhisperNode --chain-id comets-test --fees 40000ucmdx
```

<br>
