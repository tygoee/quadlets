# Technitium DNS container

Quadlet files for getting Technitium DNS working in Podman using the latest `technitium/dns-server` image from [Docker Hub](https://hub.docker.com/r/technitium/dns-server).

This configuration is made to be run as a rootless user, please adjust storage volume locations and other specifics to your needs.

## Setup

1. Adjust storage locations in the [container file](./dns-server.container)

2. Copy the `dns-server.container` file to `~/.config/containers/systemd` (or the corresponding [system directory](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path) when running rootful)

3. Reload the systemd daemon with `systemd --user daemon-reload`

4. Start the dns-server service by running `systemd --user start dns-server.service`

After forwarding port 1053 to 53 in your firewall and allowing external traffic for DNS, you now have a running Nextcloud container on port 5080. It will automatically start on every boot from now on.

With `firewalld` that is done with these commands:

```sh
sudo firewall-cmd --add-forward-port=port=53:proto=tcp:toport=1053 --permanent
sudo firewall-cmd --add-forward-port=port=53:proto=udp:toport=1053 --permanent
sudo firewall-cmd --add-rich-rule='rule family="ipv6" forward-port port="53" protocol="tcp" to-port="1053"'
sudo firewall-cmd --add-rich-rule='family="ipv6" forward-port port="53" protocol="udp" to-port="1053"'
sudo firewall-cmd --reload
```

## Sources

[DNS container via Podman - Reddit](https://www.reddit.com/r/technitium/comments/1cfdrc4/dns_container_via_podman/)

[podman-systemd.unit - systemd units using Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
