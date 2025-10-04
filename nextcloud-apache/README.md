# Nextcloud for Podman

Quadlet files for setting up Nextcloud with the Apache webserver in Podman, using the `apache` image from [Docker Hub](https://hub.docker.com/_/nextcloud/)

There is some manual configuration required, like setting database passwords and mounting disks

When running other containers as the same user you may want to prefix the names (like `nextcloud-mariadb.container`)

## Setup

1. Go through the container files and add passwords at the corresponding environment variables and change the volume paths where you want your data to be mounted

2. Place all files from the [containers](./containers) directory in `~/.config/containers/systemd` (or the corresponding [system directory](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path) when running rootful)

3. Reload the systemd daemon with `systemd --user daemon-reload`

4. Start the nextcloud-pod service by running `systemd --user start nextcloud-pod.service`

After adding port `5080/tcp` to your firewall configuration, you now have a running Nextcloud container on port 5080. It will automatically start on every boot from now on.

## Sources

[Tutorial for running Nextcloud in rootless Podman with mariadb, redis, caddy webserver - Nextcloud community](https://help.nextcloud.com/t/tutorial-for-running-nextcloud-in-rootless-podman-with-mariadb-redis-caddy-webserver-all-behind-a-caddy-reverse-proxy/159216)

[podman-systemd.unit - systemd units using Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
