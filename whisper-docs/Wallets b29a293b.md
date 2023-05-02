## Mnemonics

When you create a new key, you'll receive the mnemonic phrase that can be used to restore that key. Backup the mnemonic phrase:

```
$  secretcli keys add keyname
{
  "name": "mykey",
  "type": "local",
  "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
  "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf",
  "mnemonic": "banner genuine height east ghost oak toward reflect asset marble else explain foster car nest make van divide twice culture announce shuffle net peanut"
}
```

To restore the key:

```
$ secretcli keys add keyname-restored --recover
> Enter your bip39 mnemonic
banner genuine height east ghost oak toward reflect asset marble else explain foster car nest make van divide twice culture announce shuffle net peanut
{
  "name": "keyname-restored",
  "type": "local",
  "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
  "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
}

$ <networkd> keys list
[
  {
    "name": "keyname-restored",
    "type": "local",
    "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
    "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
  },
  {
    "name": "keyname",
    "type": "local",
    "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
    "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
  }
]
```

Note: If the mnemonics were generated using a `secretci` version of `v0.0.x` or `v0.2.x`, you'll need to use this command: `secretcli keys add keyname-restored --recover --hd-path "44'/118'/0'/0/0"`.

# Export

To backup a local key without the mnemonic phrase, backup the output of `secretcli keys export`:

```
$ secretcli keys export keyname
Enter passphrase to decrypt your key:
Enter passphrase to encrypt the exported key:
-----BEGIN TENDERMINT PRIVATE KEY-----
kdf: bcrypt
salt: 14559BB13D881A86E0F4D3872B8B2C82
type: secp256k1

3OkvaNgdxSfThr4VoEJMsa/znHmJYm0sDKyyZ+6WMfdzovDD2BVLUXToutY/6iw0
AOOu4v0/1+M6wXs3WUwkKDElHD4MOzSPrM3YYWc=
=JpKI
-----END TENDERMINT PRIVATE KEY-----

$ echo "\
-----BEGIN TENDERMINT PRIVATE KEY-----
kdf: bcrypt
salt: 14559BB13D881A86E0F4D3872B8B2C82
type: secp256k1

3OkvaNgdxSfThr4VoEJMsa/znHmJYm0sDKyyZ+6WMfdzovDD2BVLUXToutY/6iw0
AOOu4v0/1+M6wXs3WUwkKDElHD4MOzSPrM3YYWc=
=JpKI
-----END TENDERMINT PRIVATE KEY-----" > keyname.export
```

To restore the key:

```
$ secretcli keys import keyname-imported ./keyname.export
Enter passphrase to decrypt your key:

$ secretcli keys list
[
  {
    "name": "keyname-imported",
    "type": "local",
    "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
    "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
  },
  {
    "name": "keyname-restored",
    "type": "local",
    "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
    "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
  },
  {
    "name": "keyname",
    "type": "local",
    "address": "secret1zjqdn0j7fzsx5ldv0lf3ejfjkey0ce8e7vglnm",
    "pubkey": "secretpub1addwnpepqtu8pwkdft3cz65u0m84vh6k5kqmqf73lzs7we5dkx296mqlx7z6524jcxf"
  }
]
```

<br>
