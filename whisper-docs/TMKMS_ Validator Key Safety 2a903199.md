TMKMS allows us to secure our validator keys on a remote server. This provides us the benefit of double sign protection, as well as a more secure environment for our validator keys.

---

## Install Dependencies:

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
sudo apt update
sudo apt-get update -y && sudo apt upgrade -y && sudo apt-get install make build-essential gcc git jq chrony -y
sudo apt install libusb-1.0-0-dev -y
export RUSTFLAGS=-Ctarget-feature=+aes,+ssse3
```

## Install TMKMS:

```shell
cd $HOME
git clone https://github.com/iqlusioninc/tmkms.git
cd $HOME/tmkms
cargo install tmkms --features=softsign
tmkms init config
tmkms softsign keygen ./config/secrets/secret_connection_key
```

## Setup TMKMS:

```shell
mkdir ~/kms
cd ~/kms

# make chain directory
mkdir chihuahua
cd chihuahua
tmkms init .

# copy priv_validator_key to /secrets/key.json & create files
cp ~/backup-keys/priv_validator_key.json ~/kms/chihuahua/secrets/priv_validator_key.json
tmkms softsign import secrets/priv_validator_key.json secrets/validator_key.key
```

## Modify tmkms.toml:

```shell
nano tmkms.toml

# Tendermint KMS configuration file
## Chain Configuration
[[chain]]
id = "quasar-1"
key_format = { type = "bech32", account_key_prefix = "quasarpub", consensus_key_prefix = "quasarvalconspub" }
state_file = "/home/whispernode/kms/quasar/state/quasar-1-consensus.json"

## Signing Provider Configuration
### Software-based Signer Configuration
[[providers.softsign]]
chain_ids = ["quasar-1"]
key_type = "consensus"
path = "/home/whispernode/kms/quasar/secrets/validator_key.key"

## Validator Configuration
[[validator]]
chain_id = "quasar-1"
addr = "tcp://148.113.159.22:18259"
secret_key = "/home/whispernode/kms/quasar/secrets/kms-identity.key"
# this may need to be updated via {daemon} tendermint version
protocol_version = "v0.34"
reconnect = true
```

## Create a service file:

```shell
sudo nano /etc/systemd/system/tmkms-quasar.service

# unit file
[Unit]
Description=Quasar TMKMS
After=network.target

[Service]
Type=simple
User=whispernode
WorkingDirectory=/home/whispernode/
ExecStart=/home/whispernode/.cargo/bin/tmkms start -c /home/whispernode/kms/quasar/tmkms.toml
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Then, we'll enable the service file and start it:

```shell
sudo systemctl daemon-reload 
sudo systemctl enable tmkms_secret
sudo systemctl restart tmkms_secret
journalctl -u tmkms_secret -f
```

## Update Target Validator Node:

```shell
nano ~/.chihuahua/config/config.toml

# modify validator config.toml

-> priv_validator_laddr = "tcp://0.0.0.0:26659"
-> # priv_validator_key_file = "config/priv_validator_key.json"
-> # priv_validator_state_file = "data/priv_validator_state.json"

# remove validator key from node
rm ~/.comdex/config/priv_validator_key.json

# open ports
sudo ufw allow from 65.108.123.125 to any port 26659

# restart daemon
sudo systemctl restart cosmovisor 
journalctl -u cosmovisor -f
```

Note: you may need to restart `tmkms_chihuahua` on the TMKMS server after opening ports:&#x20;

```shell
sudo systemctl restart tmkms_chihuahua
journalctl -u tmkms_chihuahua -f
```

<br>
