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

In the `.profile` file, usually located at `~/.profile`, add:

```shell
# comdex cosmovisor daemon
export DAEMON_NAME_2=comdex
export DAEMON_HOME_2=$HOME/.comdex
```

Then source your profile to have access to these variables:

```shell
source ~/.profile

# ZSH
source ~/.zshrc
```

You can confirm success like so:

```shell
echo $DAEMON_NAME_2
```

It should return `comdex`.

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
mkdir -p ~/.comdex/cosmovisor/genesis/bin
mkdir -p ~/.comdex/cosmovisor/upgrades

# Change Ownership of newly created folders (if needed)
sudo chown -R whispernode:whispernode ~/.comdex/cosmovisor
```

---

## Set up genesis binary

Cosmovisor needs to know which binary to use at genesis. We put this in `$DAEMON_HOME/cosmovisor/genesis/bin`.

First, find the location of the binary you want to use:

```shell
which comdex
```

Then use the path returned to copy it to the directory Cosmovisor expects. Let's assume the previous command returned `/home/your-user/go/bin/comdex`:

```shell
cp $GOPATH/bin/comdex ~/.comdex/cosmovisor/genesis/bin
```

Once you're done, check the folder structure looks correct using a tool like `tree`.

---

## Set up service

Commands sent to Cosmovisor are sent to the underlying binary. For example, `cosmovisor version` is the same as typing `comdex version`.

Nevertheless, just as we would manage `junod` using a process manager, we would like to make sure Cosmovisor is automatically restarted if something happens, for example, an error or reboot.

First, create the service file:

```shell
sudo nano /etc/systemd/system/cosmovisor-2.service
```

Change the contents of the below to match your setup - `cosmovisor` is likely at `~/go/bin/cosmovisor` regardless of which installation path you took above, but it's worth checking.

```shell
[Unit]
Description=cosmovisor-1
After=network-online.target
[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/cosmovisor-1 start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=comdex"
Environment="DAEMON_HOME=/home/whispernode/.comdex"
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
sudo -S systemctl enable cosmovisor-2 
sudo systemctl start cosmovisor-2
```

Check it is running using:

```shell
sudo systemctl status cosmovisor-2
```

If you need to monitor the service after launch, you can view the logs using:

```shell
journalctl -u cosmovisor-2 -f
```

<br>

---

# [#](https://option-2)Option 2: Manual Service File

For each chain, we'll want to create a `systemd` unit file, so that our nodes automatically restart in the event of a node reboot or other failure.

Doing so can be accomplished by running the following command:

```shell
sudo nano /etc/systemd/system/comdex.service
```

Enter the following info into the Unit file:

```shell
[Unit]
Description=comdex daemon
After=network-online.target

[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/comdex start
Restart=always
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=comdex"
Environment="DAEMON_HOME=/home/whispernode/.comdex"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
```

Finally, we need to enable our newly created service file:

```shell
sudo systemctl daemon-reload
sudo systemctl enable comdex
sudo systemctl start comdex
journalctl -u comdex -f
```

<br>
