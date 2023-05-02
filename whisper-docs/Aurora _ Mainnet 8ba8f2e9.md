Original Guide:

[You need server with following requierements | Axelar Network](https://docs.axelar.dev/validator/external-chains/aurora "You need server with following requierements | Axelar Network")

Block Explorer: Mainnet

[Aurora Block Explorer](https://aurorascan.dev/ "Aurora Block Explorer")

---

## 1. Upgrade your server

```shell
sudo apt update && sudo apt upgrade -y
```

```shell
sudo apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen unzip -y
```

## 2. Install docker and docker-compose

Check the latest version of [docker-compose](https://github.com/docker/compose) and follow the guide

```shell
sudo apt install docker.io -y
```

```shell
git clone https://github.com/docker/compose
```

```shell
cd compose
git checkout v2.6.1
sudo make
```

```shell
cd
sudo mv compose/bin/docker-compose /usr/bin
sudo docker-compose version
```

## 3. Install and synch Aurora node

Download installation from the [official documentation](https://github.com/aurora-is-near/partner-relayer-deploy)

```shell
git clone https://github.com/aurora-is-near/partner-relayer-deploy
cd partner-relayer-deploy
```

For mainnet run:

```shell
sudo ./setup.sh
```

### 4. Run Aurora node through docker-compose

To run docker, you must be in the folder `partner-relayer-deploy`

```shell
sudo docker-compose up -d
```

To see logs, run the following:

```shell
sudo docker-compose logs -f
```

## 5. Create SSH keys on the Aurora Node

Please run the following command on the Aurora server (just press “Enter” for each question):

```shell
cd
ssh-keygen -t ed25519
```

---

## Switch to Axelar Node and run the following:

### 6. Add a public key on the Axelar Node

Once the key pair has been generated, you need to create it on the Axelar Node `/root/.ssh/authorized_keys` file and copy content from the `/root/.ssh/id_ed25519.pub` file which was created on the Aurora Node:

```shell
mkdir .ssh
nano /root/.ssh/authorized_keys
```

After this we need to set the following permissions:

```shell
sudo chmod 600 /root/.ssh/authorized_keys
```

When you perform these actions you should be able to connect from Aurora Node to Axelar Node without a password

### 7. Create SSH tunnel between Aurora Node and Axelar Main Node

On the Aurora server please run the following command to create a tunnel to forwarding ports:

```shell
ssh -f -N root@X.X.X.X -R 8545:`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}:8545{{end}}' endpoint`
```

Please note `X.X.X.X` this is the IP address of your Axelar Main Node

### 8. Check that the necessary port is listening on the Axelar Node

On the Axelar Node please run the following command to make sure that the port is listening:

```shell
netstat -atnp | grep 8545
tcp        0      0 0.0.0.0:8545            0.0.0.0:*               LISTEN      2306635/sshd: root
tcp6       0      0 :::8545                 :::*                    LISTEN      2306635/sshd: root
```

### 9. Create a script to run it automatically after reboot

Create directory

```shell
sudo mkdir /root/work
```

Create `tunnel.ssh` file:

```shell
sudo nano /root/work/tunnel.ssh
```

and add the following line into it:

```shell
ssh -f -N root@X.X.X.X -R 8545:`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}:8545{{end}}' endpoint`
```

Please note `X.X.X.X` this is the IP address of your Axelar Main Node

Then open `crontab` file and choose option 1:

```shell
crontab -e
```

and add this line to the end of it:

```shell
@reboot /root/work/tunnel.sh
```

so when the Aurora server will be rebooted the SSH tunnel will be created automatically

### 10. Connect your Aurora to Axelar

In order for Axelar Network to connect to your Aurora node, your rpc\_addr should be exposed in this format:

```shell
"http://127.0.0.1:8545"
```

<br>
