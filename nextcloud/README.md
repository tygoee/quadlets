# Additional configuration

When running other containers as the same user you might want to prefix the names (like `nextcloud-mariadb.container`)

## Create podman secrets

Create passwords for MariaDB:

```
echo -n "password" | podman secret create nextcloud_mysql_root -
echo -n "password" | podman secret create nextcloud_mysql -
```

## Open port

Open port `5080/tcp` in your firewall or add it to your reverse proxy

## NFS share

The `.volume` file uses an NFS share. NFS shares can't use idmap, so we need to map a user for the share. As seen in `nextcloud.pod`, we map www-data (33) in the container to user:group 3000:3000. This means that on the NFS server, you need to do `chown -R 3000:3000 /.../folder`. You may also need to modify other values in the `.volume` file for your setup, or just use a local volume mount.

# Sources

[Tutorial for running Nextcloud in rootless Podman with mariadb, redis, caddy webserver, all behind a caddy reverse proxy -  Support - Nextcloud community](https://help.nextcloud.com/t/tutorial-for-running-nextcloud-in-rootless-podman-with-mariadb-redis-caddy-webserver-all-behind-a-caddy-reverse-proxy/159216)

[podman-systemd.unit - Podman documentation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
