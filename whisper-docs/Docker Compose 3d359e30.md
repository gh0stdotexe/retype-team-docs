### Original Guide:

[Install Docker Compose](https://docs.docker.com/compose/install/ "Install Docker Compose")

---

## Install Compose on Linux systems

On Linux, you can download the Docker Compose binary from the [Compose repository release page on GitHub](https://github.com/docker/compose/releases). Follow the instructions from the link, which involve running the `curl` command in your terminal to download the binaries. These step-by-step instructions are also included below.

> For `alpine`, the following dependency packages are needed: `py-pip`, `python3-dev`, `libffi-dev`, `openssl-dev`, `gcc`, `libc-dev`, `rust`, `cargo` and `make`.

1. Run this command to download the current stable release of Docker Compose:
   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```
   > To install a different version of Compose, substitute `1.29.2` with the version of Compose you want to use. For instructions on how to install Compose `2.2.3` on Linux, see [Install Compose 2.0.0 on Linux](https://docs.docker.com/compose/cli-command#install-on-linux)
   If you have problems installing with `curl`, see [Alternative Install Options](https://docs.docker.com/compose/install/#alternative-install-options) tab above.
2. Apply executable permissions to the binary:
   ```shell
   sudo chmod +x /usr/local/bin/docker-compose
   ```
   > **Note:**
   >
   > If the command `docker-compose` fails after installation, check your path. You can also create a symbolic link to `/usr/bin` or any other directory in your path.
   >
   > For example:
   >
   > ```shell
   > sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
   > ```
3. Optionally, install [command completion](https://docs.docker.com/compose/completion/) for the `bash` and `zsh` shell.
4. Test the installation.
   ```shell
   sudo docker-compose --version

   # should show the following if built correctly
   docker-compose version 1.29.2, build 1110ad01
   ```

---

## Upgrading

If you’re upgrading from Compose 1.2 or earlier, remove or migrate your existing containers after upgrading Compose. This is because, as of version 1.3, Compose uses Docker labels to keep track of containers, and your containers need to be recreated to add the labels.

If Compose detects containers that were created without labels, it refuses to run, so that you don’t end up with two sets of them. If you want to keep using your existing containers (for example, because they have data volumes you want to preserve), you can use Compose 1.5.x to migrate them with the following command:

```shell
sudo docker-compose migrate-to-labels
```

Alternatively, if you’re not worried about keeping them, you can remove them. Compose just creates new ones.

```shell
sudo docker container rm -f -v myapp_web_1 myapp_db_1 ...
```

## Uninstallation

To uninstall Docker Compose if you installed using `curl`:

```shell
sudo rm /usr/local/bin/docker-compose
```

To uninstall Docker Compose if you installed using `pip`:

```shell
pip uninstall docker-compose
```

> **Got a “Permission denied” error?**
>
> If you get a “Permission denied” error using either of the above methods, you probably do not have the proper permissions to remove `docker-compose`. To force the removal, prepend `sudo` to either of the above commands and run again.
