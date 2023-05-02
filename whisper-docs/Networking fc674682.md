Various commands that can be useful when troubleshooting validator nodes running Linux.

## Find all active sessions on a VPS

```shell
ss | grep -i ssh
```

Display all proccesses running on a specific port:

```
sudo netstat -pna | grep <port>
```

Closing a terminal session properly (so as to terminate the session/not leave any hung users behind):

```
exit
```

<br>
