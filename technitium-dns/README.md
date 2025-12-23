# Additional configuration

## Add forwarding rules

It's needed to forward port `53` externally to port `1053` internally. Using `firewalld` that is done with the following commands. **Don't forward this to the wider internet, as it would be used in DNS Amplication attacks**:

```sh
sudo firewall-cmd --add-forward-port=port=53:proto=tcp:toport=1053 --permanent
sudo firewall-cmd --add-forward-port=port=53:proto=udp:toport=1053 --permanent
sudo firewall-cmd --add-rich-rule='rule family="ipv6" forward-port port="53" protocol="tcp" to-port="1053"'
sudo firewall-cmd --add-rich-rule='family="ipv6" forward-port port="53" protocol="udp" to-port="1053"'
sudo firewall-cmd --reload
```

Now, forward/redirect the following ports, or add them to your reverse proxy in the case of HTTP/HTTPS. Ports that are not shared by default will need to be uncommented in the container file.

| Port map             | Protocol   | Description                                            | Default |
|----------------------|------------|--------------------------------------------------------|---------|
| 5380                 | tcp        | Web console (HTTP)                                     | Yes     |
| 53443                | tcp        | Web console (HTTPS)                                    | No      |
| 1053:53              | tcp+udp    | DNS service                                            | Yes     |
| 1853:853             | tcp<br>udp | DNS-over-TLS service<br>DNS-over-QUIC service          | No      |
| 8443:443             | tcp<br>udp | DNS-over-HTTPS (HTTP/1.1,2)<br>DNS-over-HTTPS (HTTP/3) | No      |
| 8080:80<br>8053:8053 | tcp        | DNS-over-HTTP service<br>(for reverse proxy)           | No      |
| 1067:67              | udp        | DHCP service                                           | No      |

---

> [!NOTE]  
> In the container configuration the DNS servers are set to cloudflare. This is done so there are no issues connecting to upstream DNS providers if your local machine's DNS is set to cloudflare.

## Sources

[DnsServer/docker-compose.yml at master · TechnitiumSoftware/DnsServer](https://github.com/TechnitiumSoftware/DnsServer/blob/master/docker-compose.yml)

[DNS container via Podman : r/technitium](https://www.reddit.com/r/technitium/comments/1cfdrc4/dns_container_via_podman/)

[podman-systemd.unit - Podman documentation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
