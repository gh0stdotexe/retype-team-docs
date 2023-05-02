With Nomic, we can run into some unique situations at times. Since it's rewritten entirely in Rust, it doesn't quite follow the "standard" Tendermint convention when it comes to troubleshooting issues.

Here are some previous issues that have been encountered, and known fixes:

### Nonce Errors - Nomic Signer / Nomic services

=="ABCI Error: CheckTx failed: Nonce Error: Nonce is not valid. Expected 1074-2073, got 263"==

We can resolve this issue by running the following command, which will reset the `nonce` to a  different value

```shell
printf "%b" '\x0\x0\x0\x0\x0\x0\x1\x0' > ~/.orga-wallet/nonce 
```

<br>
