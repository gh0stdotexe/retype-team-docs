<br>

**Tenderduty Github Link:**

[GitHub - blockpane/tenderduty: Notification tool for...](https://github.com/blockpane/tenderduty "GitHub - blockpane/tenderduty: Notification tool for Cosmos/Tendermint validators, sends alerts when missing pre-commits")

**Docker Compose Install Link:**

[Install Docker Compose](https://docs.docker.com/compose/install/ "Install Docker Compose")

---

This guide will help us install Tenderduty and connect it to the PagerDuty incident response system. This will enable us to monitor validator status and ensure that we're alerted if and only when there's a true uptime emergency with a node.

Server-side installation is rather simple, with 2 major parts (Docker-Compose, and the Tenderduty app itself).

Let's begin!

# Install Compose

### Install Compose on Linux systems

On Linux, you can download the Docker Compose binary from the [Compose repository release page on GitHub](https://github.com/docker/compose/releases). Follow the instructions from the link, which involve running the `curl` command in your terminal to download the binaries. These step-by-step instructions are also included below.

> For `alpine`, the following dependency packages are needed: `py-pip`, `python3-dev`, `libffi-dev`, `openssl-dev`, `gcc`, `libc-dev`, `rust`, `cargo` and `make`.

1. *Run this command to download the current stable release of Docker Compose:*
   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```
   > To install a different version of Compose, substitute `1.29.2` with the version of Compose you want to use.
   If you have problems installing with `curl`, see [Alternative Install Options](https://docs.docker.com/compose/install/#alternative-install-options) tab above.\

2. *Apply executable permissions to the binary:*
   ```shell
   sudo chmod +x /usr/local/bin/docker-compose
   ```

> **Note**: If the command `docker-compose` fails after installation, check your path. You can also create a symbolic link to `/usr/bin` or any other directory in your path.

For example:

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

1. *Optionally, install* [*command completion*](https://docs.docker.com/compose/completion/) *for the* *`bash`* *and* *`zsh`* *shell.*
2. *Test the installation.*

```shell
sudo docker-compose --version
```

# Install Docker Engine

### Install using the repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

### Set up the repository

1. Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:
   ```shell
   sudo apt-get update -y
   sudo apt-get install ca-certificates curl gnupg lsb-release -y
   ```
2. Add Docker’s official GPG key:
   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```
3. Use the following command to set up the **stable** repository. To add the **nightly** or **test** repository, add the word `nightly` or `test` (or both) after the word `stable` in the commands below. [Learn about **nightly** and **test** channels](https://docs.docker.com/engine/install/).
   ```shell
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

## Install Docker Engine

- Update the `apt` package index, and install the *latest version* of `Docker Engine` and `containerd`, or go to the next step to install a specific version:

```shell
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

- > **Got multiple Docker repositories?**
  >
  > If you have multiple Docker repositories enabled, installing or updating without specifying a version in the `apt-get install` or `apt-get update` command always installs the highest possible version, which may not be appropriate for your stability needs.

---

- Verify that Docker Engine is installed correctly by running the `hello-world` image.

```shell
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The `docker` group is created but no users are added to it. You need to use `sudo` to run Docker commands. Continue to [Linux post-install](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands and for other optional configuration steps.

## Upgrade Docker Engine

To upgrade Docker Engine, first, run:

```shell
sudo apt-get update
```

Then follow the [installation instructions](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository), choosing the new version you want to install.

# TenderDuty

# Installing

- [Docker](https://github.com/blockpane/tenderduty/blob/main/docs/install.md#docker-container)
- [Docker Compose](https://github.com/blockpane/tenderduty/blob/main/docs/install.md#docker-compose)
- [Build From Source](https://github.com/blockpane/tenderduty/blob/main/docs/install.md#building-from-source)
- [Systemd Service](https://github.com/blockpane/tenderduty/blob/main/docs/install.md#run-as-a-systemd-service-on-ubuntu)

For this guide, we'll use the `docker compose` method of building Tenderduty, as it's quick and easy 🙂

## Docker Compose

```shell
mkdir tenderduty && cd tenderduty

cat > docker-compose.yml << EOF
---
version: '3.2'
services:

  v2:
    image: ghcr.io/blockpane/tenderduty:latest
    command: ""
    ports:
      - "8888:8888" # Dashboard
      - "28686:28686" # Prometheus exporter
    volumes:
      - home:/var/lib/tenderduty
      - ./config.yml:/var/lib/tenderduty/config.yml
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
    restart: unless-stopped

volumes:
  home:
EOF

sudo docker-compose pull
sudo docker run --rm ghcr.io/blockpane/tenderduty:latest -example-config >config.yml
```

Then, we'll edit our `config.yml` file with all node information, and then start our docker container:

```shell
sudo docker-compose up -d
sudo docker-compose logs -f --tail 20
```

## Updating Tenderduty

In order to add new nodes (or update existing node information), and have them show up on the dashboard, we'll need to first update our `config.yml` file, and then, rebuild our docker container.

```shell
sudo docker-compose kill
sudo docker system prune -a --volumes
sudo docker-compose up -d

# monitor logs to ensure it's running
sudo docker-compose logs -f --tail 20
```

<br>
