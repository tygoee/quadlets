# Additional configuration

> [!WARNING]
> This is incomplete, never really tested and does not renew on an interval. Kept in here for reference only

This assumes you're running this container as root

Before first run:

- Create a named podman volume:

```sh
sudo podman volume create certbot-creds
```

- Create a file called `cloudflare.ini` with the following contents in `/var/lib/containers/storage/volumes/certbot-creds/_data/`:

```ini
dns_cloudflare_api_token = <token>
```

- Restrict its permissions:

```sh
sudo chmod 600 /var/lib/containers/storage/volumes/certbot-creds/_data/cloudflare.ini
```

- Run the initial setup:

```sh
sudo podman run -it --rm --name certbot \
            -v letsencrypt:/etc/letsencrypt:z \
            -v letsencrypt-data:/var/lib/letsencrypt:Z \
            -v certbot-creds:/creds:Z \
            docker.io/certbot/dns-cloudflare:latest certonly \
            --dns-cloudflare-credentials /creds/cloudflare.ini
```

- For wildcard cloudflare domains, authenticate with a DNS TXT record

- For the domain names, enter `example.com,*.example.com`

# Sources

<https://eff-certbot.readthedocs.io/en/latest/install.html#running-with-docker>

<https://certbot-dns-cloudflare.readthedocs.io/en/stable/>
