<br>

Create new users for each snapshot process:

```shell
sudo useradd -m -s /usr/sbin/nologin juno
```

```shell
sudo useradd -m -s /usr/sbin/nologin comdex
```

Add new users to sudoers group w/ modified perms:

```shell
sudo visudo
```

```shell
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

Cmnd_Alias COMDEX_SERVICE = /usr/bin/systemctl * cosmovisor.service
Cmnd_Alias JUNO_SERVICE = /usr/bin/systemctl * junod.service

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
comdex    ALL=NOEXEC: NOPASSWD: COMDEX_SERVICE
juno    ALL=NOEXEC: NOPASSWD: JUNO_SERVICE
```

Next, we'll test to ensure our new users can user `sudo` without needing to enter a password - this is crucial for the snapshot script to fire properly:

```shell
sudo sudo -u comdex -s
```

<br>
