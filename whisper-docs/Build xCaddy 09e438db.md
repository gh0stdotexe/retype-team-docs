References, Guides + Sources:

[secure-server-setup-ansible/install\_caddy.yml at master ·...](https://github.com/LavenderFive/secure-server-setup-ansible/blob/master/roles/setup/tasks/install_caddy.yml "secure-server-setup-ansible/install_caddy.yml at master · LavenderFive/secure-server-setup-ansible")

[GitHub - caddyserver/xcaddy: Build Caddy with plugins](https://github.com/caddyserver/xcaddy "GitHub - caddyserver/xcaddy: Build Caddy with plugins")

<br>

<br>

We'll use `XCaddy` to harden our `caddy` installation & also build in plugins, allowing us to utilize rate-limiting and other cool features (such as Cloudflare DNS).

---

## Update System Buffer Rate (2MB)

```shell
sudo sysctl -w net.core.rmem_max=2048000
```

---

## Install + Build XCaddy

Next, we'll download & build XCaddy from source, utilizing the following GitHub repository:

[GitHub - caddyserver/xcaddy: Build Caddy with plugins](https://github.com/caddyserver/xcaddy "GitHub - caddyserver/xcaddy: Build Caddy with plugins")

### Install XCaddy

```shell
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-xcaddy.list
sudo apt update
sudo apt install xcaddy
```

### Build XCaddy with Plugins

```shell
xcaddy build --with github.com/caddyserver/transform-encoder --with github.com/RussellLuo/caddy-ext/ratelimit
```

### Move XCaddy file

```shell
sudo cp ~/caddy /usr/bin/caddy
```

---

## Caddyfile Configuration

After we've installed XCaddy, we're going to need to set our `Caddyfile` up, which will set the "rules" that Caddy will play by.

We'll have two different types of files - the master `Caddyfile`, located at `/etc/caddy/Caddyfile`, and our `chain.caddy` files, placed at `/etc/caddy/chain.caddy.`&#x20;

The main `Caddyfile` will contain our master configuration, while the `chain.caddy` files will contain individual reverse proxy configurations for each chain we wish to set up as an `RPC/REST endpoint` node.

### Caddyfile

```shell
sudo nano /etc/caddy/Caddyfile

# paste the configuration below into our Caddyfile
{
	order rate_limit before basicauth
	# to see logs -
	# tail -100f /var/log/caddy/access.log | while read -r line; do echo -e "$line"; done
	log http.log.access {
			include http.log.access
			output file /var/log/caddy/access.log
			format formatted "\e[35m[{ts}]\e[0m \e[96m\e[1m{request>remote_ip}\e[0m \e[31m{request>headers>X-Forwarded-For}\e[0m \e[33m{request>method}\e[0m \e[92m{request>host}\e[32m{request>uri}\e[0m \e[97m{status}\e[0m   \e[90m{request>headers>User-Agent}\e[0m \e[34m{request>headers>Referer}\e[0m" {
					time_format "02/Jan/2006:15:04:05-0700"
			}
	}
	log {
			exclude http.log.access
			output file /var/log/caddy/caddy.log
			format json
	}
}

(rate-limiter) {
	# checks for list of allowed IPs
	@ipfilter {
{% for allowed in white_list %}
	{% if allowed is iterable and (allowed is not string and allowed is not mapping) %}
		{% for allowed_ip in allowed %}
	not header X-FORWARDED-FOR {{ allowed_ip }}
		{% endfor %}
	{% else  %}
	not header X-FORWARDED-FOR {{ allowed }}
	{% endif  %}
{% endfor %}
	}

	handle @ipfilter {
		handle_path /cosmos/staking/v1beta1/validators/*/delegations/* {
			respond 404
		}
		handle_path /status* {
			respond 404
		}
		rate_limit {header.X-FORWARDED-FOR} 50r/m
		rate_limit {path.*} 200r/m
		rate_limit {header.X-FORWARDED-FOR} 5r/s
		rate_limit {path.*} 15r/s
	}
}

(blocked) {
# checks for blocked IPs and re-directs them to 403
	@blocked {
		header User-Agent SubQuery-Node
{% for blocked in blocked_list %}
		header X-FORWARDED-FOR {{ blocked }}
{% endfor %}
	}
	respond @blocked "<h1>Access Denied</h1>" 403
}

handle_errors {
	respond "{err.status_code} {err.status_text}"
}

:80 {
	root * /usr/share/caddy
	file_server
}

import /etc/caddy/*.caddy
```

Save the file, and then we'll create our `chain.caddy` files.

### Chain Caddyfiles

```shell
sudo nano /etc/caddy/quasar.caddy

# paste the configuration below to configure RPC/REST reverse proxies

lcd-quasar.whispernode.com {
        route /foo {
                rate_limit {remote.ip} 10r/s
                rate_limit {remote.ip} 100r/m
                respond 200
        }
        log
        reverse_proxy {
                to http://127.0.0.1:18217
        }
        header {
                Access-Control-Allow-Methods "POST, GET, OPTIONS"
                Access-Control-Allow-Headers "*"
                Access-Control-Allow-Origin "*"
                Access-Control-Max-Age 86400
                Cache-Control no-cache
                Pragma no-cache
                -Server
        }
        tls gh0st@whispernode.com
}

rpc-quasar.whispernode.com {
        route /foo {
                rate_limit {remote.ip} 10r/s
                rate_limit {remote.ip} 100r/m
                respond 200
        }
        log
        reverse_proxy {
                to http://127.0.0.1:18257
        }
        header {
                Access-Control-Allow-Methods "POST, GET, OPTIONS"
                Access-Control-Allow-Headers "*"
                Access-Control-Allow-Origin "*"
                Access-Control-Max-Age 86400
                Cache-Control no-cache
                Pragma no-cache
                -Server
        }
        tls gh0st@whispernode.com
}
```

Each chain will get its own `chain.caddy` file; multiple can be created on the same node, allowing us to create a multi-endpoint node.

---

## Update Service File

Last thing we need to do is to update our `caddy.service` file with some tweaks, to further optimize our setup:

Configure the service file in the following way:

```shell
sudo nano /lib/systemd/system/caddy.service

# copy this configuration
# caddy.service
#
# For using Caddy with a config file.
#
# Make sure the ExecStart and ExecReload commands are correct
# for your installation.
#
# See https://caddyserver.com/docs/install for instructions.
#
# WARNING: This service does not use the --resume flag, so if you
# use the API to make changes, they will be overwritten by the
# Caddyfile next time the service is restarted. If you intend to
# use Caddy's API to configure it, add the --resume flag to the
# `caddy run` command or use the caddy-api.service file instead.

[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile --force
TimeoutStopSec=5s
RuntimeMaxSec=7200
Restart=always
LimitNOFILE=1048576
LimitNPROC=512
PrivateDevices=yes
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

<br>
