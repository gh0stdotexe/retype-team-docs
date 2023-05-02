## Key based SSH authentication

SSH keys, similarly to crypto currency keys, consist of a public and private key. The machine you store the private key on should be a machine you trust, and the correspending public key is what you add to your server to secure it. **For this reasons be sure to securely store a backup of your private ssh key.**

From your local machine that you plan to SSH from, generate an SSH key. This is likely going to be your laptop or desktop computer. This is written for OSX or Linux.

```shell
ssh-keygen -t ecdsa
```

Decide on a name for your key and proceed through the prompts.

```shell
Your identification has been saved in /Users/myname/.ssh/id_ecdsa.
Your public key has been saved in /Users/myname/.ssh/id_ecdsa.pub
The key fingerprint is:
ae:89:72:0b:85:da:5a:f4:7c:1f:c2:43:fd:c6:44:38 myname@mymac.local
The key's randomart image is:
+--[ ECDSA 256]---+
|                 |
|         .       |
|        E .      |
|   .   . o       |
|  o . . S .      |
| + + o . +       |
|. + o = o +      |
| o...o * o       |
|.  oo.o .        |
+-----------------+
```

<br>
