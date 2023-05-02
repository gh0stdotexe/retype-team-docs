For mainnet, it's recommended to use Cosmovisor to run your node. If you've not used it before, then run it during a testnet to ensure you can set it up correctly.

Setting up Cosmovisor is relatively straightforward. However, it does expect certain environment variables and folder structures to be set.

Cosmovisor allows you to download binaries ahead of time for chain upgrades, meaning that you can do zero (or close to zero) downtime chain upgrades. It's also useful if your local timezone means that a chain upgrade will fall at a bad time.

Rather than having to do stressful ops tasks late at night, it's always better if you can automate them away, and that's what Cosmovisor tries to do.

Alternatively, we can set up a manual service file.

Option 1: Cosmovisor Install

Option 2: Manual Service File

---

# Option 1: Cosmovisor Install

First, go and get cosmovisor (recommended approach):

```shell
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

Your installation can be confirmed with:

```shell
which cosmovisor
```

This will return something like:

```shell
/home/<your-user>/go/bin/cosmovisor
```

---

## Add environment variables to your shell

In the `.zshrc` file, usually located at `~/.zshrc`, add:

```shell
# defund cosmovisor daemon
export DAEMON_NAME_3=marsd
export DAEMON_HOME_3=$HOME/.mars
```

Then source your profile to have access to these variables:

```shell
source ~/.zshrc
```

You can confirm success like so:

```shell
echo $DAEMON_NAME_3
```

It should return `marsd`.

---

## Set up the folder structure

Cosmovisor expects a certain folder structure:

```shell
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── $DAEMON_NAME
└── upgrades
    └── <name>
        └── bin
            └── $DAEMON_NAME
```

Don't worry about `current` - that is simply a symlink used by Cosmovisor. The other folders will need setting up, but this is easy:

```shell
mkdir -p ~/.mars/cosmovisor/genesis/bin
mkdir -p ~/.mars/cosmovisor/upgrades

# Change Ownership of newly created folders (if needed)
sudo chown -R whispernode:whispernode ~/.mars/cosmovisor
```

---

## Set up genesis binary

Cosmovisor needs to know which binary to use at genesis. We put this in `$DAEMON_HOME/cosmovisor/genesis/bin`.

First, find the location of the binary you want to use:

```shell
which masrd
```

Then use the path returned to copy it to the directory Cosmovisor expects. Let's assume the previous command returned `/home/your-user/go/bin/marsd`:

```shell
cp $GOPATH/bin/marsd ~/.mars/cosmovisor/genesis/bin
```

Once you're done, check the folder structure looks correct using a tool like `tree`.

---

## Set up service

Commands sent to Cosmovisor are sent to the underlying binary. For example, `cosmovisor version` is the same as typing `defundd version`.

Nevertheless, just as we would manage `defundd` using a process manager, we would like to make sure Cosmovisor is automatically restarted if something happens, for example, an error or reboot.

First, create the service file:

```shell
sudo nano /etc/systemd/system/cosmovisor-5.service
```

Change the contents of the below to match your setup - `cosmovisor` is likely at `~/go/bin/cosmovisor` regardless of which installation path you took above, but it's worth checking.

```shell
[Unit]
Description=cosmovisor-5 | Mars Protocol
After=network-online.target
[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/cosmovisor-5 start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=marsd"
Environment="DAEMON_HOME=/home/whispernode/.mars"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
```

Note also that we set buffer size explicitly because of a [live bug in Cosmovisor](https://github.com/cosmos/cosmos-sdk/pull/8590) before version `v1.0.0`. If you are using `v1.0.0`, you may omit that line.

In addition, the same issue can be fixed by reducing the log via the `env` variable. If you are unsure, ask on Discord.

---

## Start Cosmovisor

Finally, enable the service and start it.

```shell
sudo -S systemctl daemon-reload 
sudo -S systemctl enable cosmovisor-5
sudo systemctl start cosmovisor-5
```

Check it is running using:

```shell
sudo systemctl status cosmovisor-3
```

If you need to monitor the service after launch, you can view the logs using:

```shell
journalctl -u cosmovisor-3 -f
```

<br>

---

# [#](https://option-2)Option 2: Manual Service File

For each chain, we'll want to create a `systemd` unit file, so that our nodes automatically restart in the event of a node reboot or other failure.

Doing so can be accomplished by running the following command:

```shell
sudo nano /etc/systemd/system/defundd.service
```

Enter the following info into the Unit file:

```shell
[Unit]
Description=defundd daemon
After=network-online.target

[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/defundd start
Restart=always
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=defundd"
Environment="DAEMON_HOME=/home/whispernode/.defund"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
```

Finally, we need to enable our newly created service file:

```shell
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl start defundd
journalctl -u defundd -f
```

<br>
