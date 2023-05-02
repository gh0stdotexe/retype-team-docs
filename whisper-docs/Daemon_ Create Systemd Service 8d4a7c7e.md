For each chain, we'll want to create a `systemd` unit file, so that our nodes automatically restart in the event of a node reboot or other failure.

Doing so can be accomplished by running the following command:

```shell
sudo nano /etc/systemd/system/defundd.service
```

Enter the following info into the Unit file:

```shell
[Unit]
Description=Injective Daemon
After=network-online.target

[Service]
User=whispernode
ExecStart=/home/whispernode/go/bin/injectived start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=injectived"
Environment="DAEMON_HOME=/home/whispernode/.injectived"
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
