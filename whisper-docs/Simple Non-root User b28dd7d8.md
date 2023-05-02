Commands that are useful for adding users, updating passwords, etc.

<br>

- Add new user w/ Sudo permissions:

```shell
adduser <username>
usermod -aG sudo <username>
```

- Test new user / switch to root permissions:

```shell
su - <username>
sudo ls -la /root
```

<br>
