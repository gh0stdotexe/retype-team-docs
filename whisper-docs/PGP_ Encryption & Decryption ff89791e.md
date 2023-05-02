PGP can be used to encrypt, store, and transmit sensitive data while making use of the latest in cryptographic security. This is incredibly important to learn and is relatively simple to pick up with some practice. Integrating PGP encryption into your everyday life can have a large impact on your overall security and privacy.

In our organization, we'll use it to encrypt our validator private keys and other sensitive data, to transmit back and forth between team members across insecure channels (Discord, etc).:\


Resource: [GPG Keys Cheatsheet](https://rtcamp.com/tutorials/linux/gpg-keys/ "GPG Keys Cheatsheet")

---

## Basic Steps:

1. `gpg --import pgp_pub.key`
2. `gpg --generate-key`
3. `gpg --export -a "brendan" > brendan_public.key`
4. `gpg -e -u "brendan.kittredge@gmail.com" -r "ben@whispernode.com" `/Users/brendan/Desktop/SentinelMnemonic.txt
5. `gpg --list-keys`

---

## Example:

Encrypting a document:

```shell
gpg -e -u "Sender (Your) Real Name" -r "Receiver User Name" file.txt
```

Decrypting a document:

```shell
gpg -d encryptfile1.txt.gpg
```

<br>
