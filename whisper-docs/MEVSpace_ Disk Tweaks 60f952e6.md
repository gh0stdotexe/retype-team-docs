Guide for Smartctl:

[How to Install and Configure Smartctl on Ubuntu](https://linuxhint.com/install-and-configure-smartctl-on-ubuntu/ "How to Install and Configure Smartctl on Ubuntu")

---

We need to update our MEVSpace disks so that they aren't "throttled" - by default, they're not configured for performance, which can cause some issues for us.

First, we need to install `smartctl`:

```shell
sudo apt install smartmontools -y
```

First, let's check the current settings:

```shell
sudo smartctl --all -P show /dev/nvme1n1 | grep "Temperature Time"
```

It checks the state of the Samsung 980 NVMe disks - they're falsely being throttled by temp, even if the temp doesn't nearly hit the throttle point. you WANT:

```shell
Warning  Comp. Temperature Time:    0
Critical Comp. Temperature Time:    0
```

If you receive anything other than this, you'll need to update the `grub` a bit.&#x20;

```shell
sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="nvme_core.default_ps_max_latency_us=1500"

# save + update
sudo update-grub
```

<br>
