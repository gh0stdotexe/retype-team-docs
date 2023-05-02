We'll use crontab to create an auto-claim and auto-stake script - this will allow us to easily restake and claim commissions & staking rewards for higher compounding rewards.

First, we'll create our script:

```shell
mkdir -p ~/scripts && cd ~/scripts
sudo nano autostake.sh
```

Then, we'll add the following lines to our script:

```shell
## nomic script

#!/bin/bash
nomic claim && nomic delegate nomic1chhhfn786l2pyy8xxf9yywh2hu7wnrztwkhlzm 1400000

## traditional tendermint script

#!/bin/bash
chihuahuad tx distribution withdraw-rewards chihuahuavaloper1m2dxkn94t97m7ah65hv4tg2xu0j2a2k4fdee20 --commission --from WhisperNode --chain-id chihuahua-1 --fees 5000uhuahua;
chihuahuad tx staking delegate chihuahuavaloper1m2dxkn94t97m7ah65hv4tg2xu0j2a2k4fdee20 1000000000uhuahua --from WhisperNode --chain-id chihuahua-1 --fees 5000uhuahua
```

Change the permissions of our new script so that it's executable:

```shell
chmod +x ~/scripts/autostake.sh
```

Lastly, we need to set up our `crontab` so that the script runs automatically every X minutes:

```shell
crontab -e
```

Paste the last line of the code below into the crontab. This script will run every `30` minutes. Change that number to whatever you want the interval (in minutes) to be. For cheap networks (like Chihuahua), faster is *usually* better.

```shell
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
*/30 * * * * ~/scripts/autostake.sh > ~/scripts/logs/autostake.log 2>&1
```

You're done!
