It is **CRUCIAL** to backup your validator's private key. It's the only way to restore your validator in an event of a disaster.

The validator private key is a Tendermint Key - a unique key used to sign consensus votes.

To backup everything you need to restore your validator, simply do the following

If you are using the software sign (which is the default signing method of tendermint), your Tendermint Key is located in `~/.<network>/config/priv_validator_key.json`.

1. Backup `~/.<network>/config/priv_validator_key.json`.
2. Backup the self-delegator wallet. See the [**wallet section**](https://app.nuclino.com/t/b/b29a293b-a0db-4cb5-84b0-9aafa74e2c77 "Nuclino").

The easiest way is to backup the whole config folder.

Also see [**Migrate a Validator**](https://app.nuclino.com/WhisperNode/Validator-Nodes/Migrate-a-Validator-60f635a5-6ff1-45c9-91fd-df95d9843216 "Nuclino").

To see the associated public key:

```shell
<networkd> tendermint show-validator
```

To see the associated bech32 address:

```shell
<networkd> tendermint show-address
```

Or you can use hardware to manage your Tendermint Key much more safely, such as [**YubiHSM2**](https://developers.yubico.com/YubiHSM2/).
