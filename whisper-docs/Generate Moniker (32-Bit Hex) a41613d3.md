We can generate a random 32-bit `hex` address for our node monikers by using the following command:

```shell
openssl rand -hex 16
```

The above command generate a random 16 byte (32 hex symbols) value and encodes it in hex (`-base64` is also supported).
