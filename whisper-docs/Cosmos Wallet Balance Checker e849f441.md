Original Guide:

[GitHub - gh0stdotexe/cosmos-balance-bot: Query...](https://github.com/gh0stdotexe/cosmos-balance-bot "GitHub - gh0stdotexe/cosmos-balance-bot: Query notifications for balances on wallets with threasholds")

# cosmos-balance-bot

A useful little bot for Relayers, Validators using restake.app, Akash Escrow Accounts, hot wallets, and anything else you want to track!

## Install on a system

```shell
git clone https://github.com/Reecepbcups/cosmos-balance-bot
cd cosmos-balance-bot

# install python
sudo apt-get install python3-pip -y

# Ensure you have python installed first. May just be `python` depending on your system
python3 -m pip install --no-cache-dir -r requirements/requirements.txt

# Edit secrets.json to your values, wallets, thresholds, etc.

python3 src/cosmos-balance-bot.py
```

<br>

## Setup:

```shell
nano secrets.json

NOTIFY_GOOD_BALANCES:(default: false)
    - When `true`, every time the bot is run it will post for all wallets even if the account is good
    - When `false`, only notifications will post if the wallet is in warning or low funds

SIMPLIFY_UDENOM_VALUES_TO_READABLE: (default: true)
    - When `true` the bot will simplify the udenom values to readable values like "juno = 1" instead of "ujuno = 1000000"
    (If you set this to false, ensure you also change all wallet denoms too.)

WALLETS: dict{dict{"status": float64}}
    - Each section specifies a wallets public address warning and low level thresholds
    - Depending on the section, the notification title & color will change in relation to the warning

TWITTER:
- Keys & Secrets to connect to a twitter bot account. This requires a developer.twitter.com account

Discord:
- Webhook URL (channel settings -> Integrations -> webhook -> create new & copy URL)
- Username: What you want the webhook bot to be called
- Accounts to tag - doesn't work yet

```

<br>
