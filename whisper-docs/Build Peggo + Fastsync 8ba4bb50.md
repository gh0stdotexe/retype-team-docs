GitHub Repo:

[GitHub - InjectiveLabs/peggo: Peggo is a Go implementation...](https://github.com/InjectiveLabs/peggo "GitHub - InjectiveLabs/peggo: Peggo is a Go implementation of the Peggy Orchestrator for the Injective Chain")

Resources (troubleshooting):

<https://ethereum.stackexchange.com/questions/97812/hidapi-hidapi-hidapi-h-no-such-file-or-directory>

[Golang checksum error](https://community.temporal.io/t/golang-checksum-error/3171/3 "Golang checksum error")[3](https://community.temporal.io/t/golang-checksum-error/3171/3)

After we build + sync our Injective validator node, we'll need to setup the Peggy relayer/oracle. Otherwise, our node will be slashed - we have to submit Peggy votes within 25000 blocks to prevent slashing.

```shell
git clone https://github.com/InjectiveLabs/peggo peggo
cd peggo
```

Then, we need to update a few files to enable "fast-syncing" of our Peggo relayer (otherwise, it'll take weeks to sync up to the current `nonce`):

```shell
nano orchestrator/cosmos/broadcast.go

# update the following setting:
SyncBroadcastMsg: change to AsyncBroadcastMsg (bypassing the 40 second timeout)
```

Save that file after editing all the `SyncBroadMsg` entries to `AsyncBroadcastMsg`. Then, edit our next file:

```shell
nano orchestrator/eth_event_watcher.go

# update the following setting:
const defaultBlocksToSearch = 1000
```

Save this file. Finally, edit the last file:

```shell
nano orchestrator/main_loops.go

# update the following setting:
const defaultLoopDur = 6 * time.Second
```

Save the file, then run the following:

```shell
go mod vendor
make install
```

If the `peggo.service` file is running, stop it. Then, copy over our newly built `peggo` binary to the `/usr/bin` directory:

```shell
sudo systemctl stop peggo
sudo rm -rf /usr/bin/peggo
sudo cp ~/go/bin/peggo /usr/bin/peggo
sudo systemctl restart peggo && journalctl -fu peggo

# you should now see the following if it was successful:
Mar 18 20:21:15 injective-0 systemd[1]: Stopped peggo.
Mar 18 20:22:04 injective-0 systemd[1]: Started peggo.
Mar 18 20:22:05 injective-0 bash[199082]: time="2023-03-18T20:22:05Z" level=info msg="Using Cosmos ValAddress inj1xxcq8g74auk6wp7wzuv93gt4agu8wkru2u445t"
Mar 18 20:22:05 injective-0 bash[199082]: time="2023-03-18T20:22:05Z" level=info msg="Using Ethereum address 0x851a9E19F3a6Cd8D08879c89E1763d80670a2490"
Mar 18 20:22:05 injective-0 bash[199082]: time="2023-03-18T20:22:05Z" level=info msg="chain session cookie loaded from disk" module=sdk-go svc=chainClient
Mar 18 20:22:05 injective-0 bash[199082]: time="2023-03-18T20:22:05Z" level=info msg="Waiting for injectived GRPC"
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Connected to Ethereum RPC at https://mainnet.infura.io/v3/d2f0fddd068b47baabccda974567599a"
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Starting peggo in validator mode"
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Valset Relay Enabled. Starting to relay valsets to Ethereum" loop=RelayerMainLoop
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Start scanning for events" lastCheckedBlock=16543989 loop=EthOracleMainLoop
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Oracle observed a withdraw batch event. Sending MsgWithdrawClaim" event_nonce=10093 nonce=3359 token_contract=0xdAC17F958D2ee523a2206206994597C13D831ec7
Mar 18 20:22:06 injective-0 bash[199082]: time="2023-03-18T20:22:06Z" level=info msg="Oracle sent Withdraw event successfully" event_nonce=10093 txHash=A900DDC3552318B577B7BA52BD5C9C2E99C5BF2AADA6A53F96897639F289A7C3
```

<br>
