Original Documentation:

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/ "Install Docker Engine on Ubuntu")

### Install using the repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

### Set up the repository

1. Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:
   ```shell
   sudo apt-get update
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

- Update the `apt` package index, and install the *latest version* of Docker Engine and containerd, or go to the next step to install a specific version:

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

- > **Got multiple Docker repositories?**
  >
  > If you have multiple Docker repositories enabled, installing or updating without specifying a version in the `apt-get install` or `apt-get update` command always installs the highest possible version, which may not be appropriate for your stability needs.

---

- **OPTIONAL**: To install a *specific version* of Docker Engine, list the available versions in the repo, then select and install:
  - a. List the versions available in your repo:
  ```shell
  apt-cache madison docker-ce

  docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  ```
  - Install a specific version using the version string from the second column, for example, `5:18.09.1~3-0~ubuntu-xenial`.
  ```shell
  sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
  ```

---

- Verify that Docker Engine is installed correctly by running the `hello-world` image.

```shell
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The `docker` group is created but no users are added to it. You need to use `sudo` to run Docker commands. Continue to [Linux postinstall](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands and for other optional configuration steps.

## Upgrade Docker Engine

To upgrade Docker Engine, first run:

```shell
sudo apt-get update
```

Then follow the [installation instructions](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository), choosing the new version you want to install.
