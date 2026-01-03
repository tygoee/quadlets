# Additional configuration

> [!NOTE]  
> This configuration is untested as some values have been changed and I'm not running this container anymore.

When running other containers as the same user you might want to prefix the names (like `nextcloud-mariadb.container`)

## Create podman secrets

Create passwords for MariaDB:

```
echo -n "msql root password" | podman secret create nextcloud_mysql_root -
echo -n "msql nextcloud password" | podman secret create nextcloud_mysql -
```

## Open port

Open port `5080/tcp` in your firewall or add it to your reverse proxy

# Sources

[Tutorial for running Nextcloud in rootless Podman with mariadb, redis, caddy webserver, all behind a caddy reverse proxy -  Support - Nextcloud community](https://help.nextcloud.com/t/tutorial-for-running-nextcloud-in-rootless-podman-with-mariadb-redis-caddy-webserver-all-behind-a-caddy-reverse-proxy/159216)

[podman-systemd.unit - Podman documentation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)

[podman-volume-ls - Podman documentation](https://docs.podman.io/en/v4.4/markdown/podman-volume-ls.1.html)
