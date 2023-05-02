# Check hardware tool

## How to run

Download Hardware Check Tool:

\
`wget `<https://github.com/scrtlabs/SecretNetwork/releases/download/v1.5.1/check-hw.tar.gz>

\
Extract `check-hw.tar.gz`:

```shell
sudo tar -xzf check-hw.tar.gz
```

Run the `check-hw` tool:

```shell
cd check-hw/
./check-hw
```

If your machine is compatible and patched, you should get:

```shell
Looking for the enclave file in Some("./librust_cosmwasm_enclave.signed.so")

...

Platform Ok!
```

<br>
