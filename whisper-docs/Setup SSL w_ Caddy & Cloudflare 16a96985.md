By default, our Tenderduty install does NOT have a valid SSL certificate - so we won't have `https` by default. We can easily fix this by using Caddy on our node, and creating a Cloudflare DNS record.

1. Install Caddy on your Tenderduty server. You can find the installation instructions for Caddy on their website.
2. Create a Caddyfile in the directory where Caddy is installed. This file will contain the configuration settings for Caddy.
3. Edit the Caddyfile and add the following lines:

```shell
monitoring.whispernode.com {
    tls {
        dns cloudflare
    }
    reverse_proxy localhost:8888
}
```

`your-domain.com { tls { dns cloudflare } reverse_proxy localhost:8888 }`

Replace **`your-domain.com`** with your actual domain name. The **`tls`** directive tells Caddy to obtain an SSL certificate for your domain using the Cloudflare DNS challenge.

4. Save the Caddyfile and start Caddy using the following command:

```shell
sudo caddy start
```

5. Log in to your Cloudflare account and navigate to the DNS settings for your domain.
6. Add a new A record for your domain and set the value to the IP address of your Tenderduty server.
7. Wait for the DNS changes to propagate (this may take up to 24 hours).
8. Once the DNS changes have propagated, navigate to [**`https://your-domain.com`**](https://your-domain.com) in your web browser. You should see the Tenderduty web interface with SSL enabled.

That's it! Your Tenderduty node is now accessible over HTTPS using a valid SSL certificate obtained from Cloudflare.
