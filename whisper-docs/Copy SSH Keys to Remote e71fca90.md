## Using ssh-copy-id utility

The `ssh-copy-id` utility is by default available on your Ubuntu machine which copies the public key from your device to the appropriate directory of your remote Ubuntu machine.

To copy the public ssh key simply type the command in your terminal, as shown below.

```shell
ssh-copy-id -i pubkeyname username@hostname
```

Replace the `username` and `hostname` in the above command with the username and host-name of your server.

The following message will appear on your terminal if you are connecting to your host for the first time, type `yes` and press `Enter`.

```shell
The authenticity of host' 172.105.XX.XX (172.105.XX.XX)' can't be established.
ECDSA key fingerprint is xx:xx:xx:xx:77:fe:73:xx:xx:55:00:ad:d6:xx:xx:xx.
Are you sure you want to continue connecting (yes/no)? yes
```

Now the `ssh-copy-id` utility will scan for the file with the name `id_rsa.pub` which contains the public SSH key. Once the scanning process is complete, it will prompt you to enter the password of your remote server, as shown below. Type the password and hit `Enter`.

```shell
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.105.XX.XX's password:
```

Once the key is added, the following message will appear on your terminal as output.

```shell
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'root@172.105.XX.XX'" and check to make sure that only the key(s) you wanted were added.In case you have multiple SSH keys on your client computer then to copy the appropriate public key to your remote computer type the command in the pattern shown below.
```

In case you have multiple SSH keys on your client computer then to copy the appropriate public key to your remote computer type the command in the pattern shown below.

```shell
ssh-copy-id -i id_rsa_xxx.pub username@host
```

<br>
