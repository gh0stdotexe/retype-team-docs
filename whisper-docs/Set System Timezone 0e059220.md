We can easily ensure that our host is using `UTC` as the timezone by running the following command:

```shell
timedatectl
```

If the output of the above command isn't showing `UTC` as the timezone used, we can tweak this by running:

```shell
sudo timedatectl set-timezone UTC
```

This will change your timezone to UTC, system-wide.

You can also use:

```shell
timedatectl list-timezones
```

to see all the available timezones.
