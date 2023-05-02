Original Guide:

[Export Logs to a File with Journalctl - Putorius](https://www.putorius.net/export-logs-to-a-file-with-journalctl.html "Export Logs to a File with Journalctl - Putorius")

---

## Export All Logs with Journalctl

If you want to just dump all the logs, you can do a simple redirection. If you are unfamiliar with the concept of redirection read our primer "[I/O, Standard Streams, and Redirection](https://www.putorius.net/linux-io-file-descriptors-and-redirection.html#redirection-operators)".

You can export all logs from journalctl like so:

```shell
[savona@putor ~]$ sudo journalctl > all_logs.txt
```

Now that you have all the logs in a text file, you can manipulate that file any way you like. However, be prepared for a rather large text file (mine was 378M). That much raw data in a single text file is daunting. Let's take a look at how we can refine this output.

## Export Specific Logs from Journalctl

In our [Viewing Logs with Journalctl](https://www.putorius.net/viewing-logs-with-journalctl-in-red-hat.html) primer, we show you how to refine your searches with journalctl. You can use any of those options, or a combination of those options to export only the logs you want.

For example, let's say we wanted to find all the NetworkManager logs since yesterday. We can use the `-u` option to specify the unit and the `-S` option to specify a start time.

NOTE: `-S` is the same as `--since` and you can also use `-U` or `--until` to get logs up to a certain date.

In the below example we are pulling all the logs for the NetworkManager service since yesterday.

```shell
[savona@putor ~]$ sudo journalctl -u NetworkManager -S yesterday > NetworkManager_logs_1day.txt
```

## Formatting the Output of Journalctl for Export

Journalctl offers many ways to format journal entries to fit your needs. For example, you can export all the logs into JSON format. You can also use the "short" option which provides classic syslog style logs, one line per entry.

### Formatting Output of Journalclt in JSON format

To format journal entries as JSON objects, use the `-o` or `--output` option and specify json as the output. Here is an example of exporting the NetworkManager Logs in JSON format.

```shell
[savona@putor ~]$ sudo journalctl -u NetworkManager --output=json > NetworkManager-JSON
[savona@putor ~]$ head NetworkManager-JSON
{"_SOURCE_REALTIME_TIMESTAMP":"1589046129512828","TIMESTAMP_BOOTTIME":"31.291066","_TRANSPORT":"journal","_SYSTEMD_INVOCATION_ID":"40e94caea07a416a8e9c5a5e5f3428d8","_PID":"952","SYSLOG_IDENTIFIER":"NetworkManager","_SELINUX_CONTEXT":"system_u:system_r:NetworkManager_t:s0"
...OUTPUT TRUNCATED...
```

### Formatting Journalctl Output in Old Syslog Format

Here is an example of formatting the journal entries into a classic syslog-style log file.

```shell
[savona@putor ~]$ sudo journalctl -u NetworkManager --output=short > NetworkManager.log
[savona@putor ~]$ head NetworkManager.log 
-- Journal begins at Wed 2020-04-01 17:26:10 EDT, ends at Sat 2021-10-16 09:01:10 EDT. --
May 09 13:42:09 putor NetworkManager[952]: <info>  [1589046129.5125] agent-manager: agent[53f749345a6757b0,:1.70/org.gnome.Shell.NetworkAgent/1000]: agent registered
May 09 14:09:13 putor NetworkManager[952]: <info>  [1589047753.9244] agent-manager: agent[36566614e251a8fc,:1.330/org.gnome.Shell.NetworkAgent/1000]: agent registered
...OUTPUT TRUNCATED...
```

There are quite a few ways to format the output of journal entries. See the man page link below for more information.
