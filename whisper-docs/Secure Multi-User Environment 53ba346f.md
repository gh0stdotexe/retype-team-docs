For creating multiple users while also restricting root access, this simple script automates the following:

- creating a user
- making the `.ssh` directory
- forcing user to set a password upon login
- providing the user`sudo` access

Build the script by making a new file `create-user.sh` :

```shell
sudo nano create-user.sh
```

Save the following within:

```shell
#!/bin/bash

#This is meant to be run as root

username=$1

if [ "$username" == "" ]; then
    echo "Username required at position 1."
    exit 1
fi


#Create the user
adduser "$username"

#Create .ssh dir
mkdir "/home/$username/.ssh"
touch "/home/$username/.ssh/authorized_keys"
chmod 600 "/home/$username/.ssh/authorized_keys"
chown -R "$username":"$username" "/home/$username/.ssh"

#Force user to set password on login
passwd -d "$username"
chage -d 0 "$username"

#Give user sudo access
usermod -aG adm "$username"
usermod -aG systemd-journal "$username"
usermod -aG sudo "$username"

echo "You still need to add the users public key to /home/$username/.ssh/authorized_keys"
```

Once the file has been created, we'll create our new user:

```shell
sudo bash create-user.sh <username>
```

Lastly, add the users public key to their `/home/$username/.ssh/authorized_keys` directory.

<br>
