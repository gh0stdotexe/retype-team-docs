As root, you have to move the file from root's homedir to *\<users>* homedir and give it ownership to *\<user>:*

```shell
# sudo su into root shell
sudo su root

# copy root file to users home directory
cp .folder/filepath ~<username>/

# exit root shell
exit
sudo chown <username>:<username> <filename>
```

<br>
