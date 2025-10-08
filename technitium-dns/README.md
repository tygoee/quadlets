# Additional configuration

## Add forwarding rules

It's needed to forward port `53` externally to port `1053` internally. `firewalld` that is done with these commands:

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
