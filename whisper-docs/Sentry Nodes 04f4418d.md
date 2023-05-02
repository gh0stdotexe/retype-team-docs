### Make your node resilient to DDoS attacks!

The Sentry Node Architecture is an infrastructure example for DDoS mitigation on Tendermint-based networks.

[**Sentry Node Architecture Overview.**](https://forum.cosmos.network/t/sentry-node-architecture-overview/454)

Secret Nodes (Validators) are responsible for ensuring that the network can sustain denial of service attacks.

One recommended way to mitigate these risks is for validators to carefully structure their network topology in a so-called sentry node architecture.

Validator nodes should only connect to full-nodes they trust because they operate them themselves or are run by other validators they know socially. A validator node will typically run in a data center. Most data centers provide direct links the networks of major cloud providers. The validator can use those links to connect to sentry nodes in the cloud. This shifts the burden of denial-of-service from the validator's node directly to its sentry nodes, and may require new sentry nodes be spun up or activated to mitigate attacks on existing ones.

Sentry nodes can be quickly spun up or change their IP addresses. Because the links to the sentry nodes are in private IP space, an internet based attacked cannot disturb them directly. This will ensure validator block proposals and votes always make it to the rest of the network.

## Notes:

For those implementing Sentries on Validators who already have Public IP exposed. Currently any peer, be it a validator or full node, is given 16 attempts with exponential backoff, which in total amounts to around 35 hours, to connect. If the node remains unreachable then it is automatically removed from the address book. An unreachable validator node is not gossiped across the network i.e. all other nodes will each try to connect to the unreachable validator node before removing it from their address book.

---

## Get your node peer ids

Log into your sentry node(s), and validator, then run the following commands to get the peer information:

Get node id:

```shell
secretd tendermint show-node-id
```

```shell
ip r
```

Save your peer information, be sure to remember which are for sentries and which is for your validator, you'll need it later:

```
<your-node-id>@<your-public-ip>:26656
```

---

## Setup Sentry Node

To setup basic sentry node architecture you can follow the instructions below:

Sentry Nodes should edit their config.toml:

First follow the [**Full Node Guide**](https://app.nuclino.com/WhisperNode/Validator-Nodes/Full-Node-Prep-7eeb60f4-d35b-48af-8d32-f886f52e52c3 "Nuclino")**.**

Edit the full nodes config file you want to use as a sentry node:

```
nano /.secretd/config/config.toml
```

Proceed to add the peer id of your validator to the `.secretd/config/config.toml`:

```
# Comma separated list of peer IDs to keep private (will not be gossiped to other peers)
# Example ID: 3e16af0cead27979e1fc3dac57d03df3c7a77acc@1.4.7.7:26656
private_peer_ids = "node_ids_of_private_peers"
```

Now proceed to restart your secret node with the following command.

```
sudo systemctl restart secret-node
```

You now have an sentry node running!

---

## Place Validator behind Sentry Nodes

Validators nodes should add their sentry node peer information to their `.secretd/config/config.toml`:

```
nano .secretd/config/config.toml
```

Proceed to add the peer id of your sentry nodes to the persistent\_peers list and set pex to false:

```
# Comma separated list of nodes to keep persistent connections to
# Do not add private peers to this list if you don't want them advertised
persistent_peers =[list of sentry nodes]

# Set true to enable the peer-exchange reactor
pex = false
```

Now proceed to restart your secret node with the following command:

```
sudo systemctl restart secret-node
```

You're now running your validator behind a sentry node!

### Resources:

[**https://github.com/cosmos/gaia/blob/master/docs/validators/security.md**](https://github.com/cosmos/gaia/blob/master/docs/validators/security.md)
