# Additional configuration

Add these ports on the firewall, and access the webserver on port `8443`. It will automatically start on every boot from now on.

| Port(s)     | Protocol | Description                                                         |
| ----------- | -------- | ------------------------------------------------------------------- |
| 8443        | tcp      | Crafty web panel                                                    |
| 8123        | tcp      | Dynmap (optional)                                                   |
| 19132       | udp      | Minecraft Bedrock Edition server                                    |
| 25500-25600 | tcp      | Minecraft Java Edition servers (only use 25565 for a single server) |

## Sources

[Docker - Crafty Documentation](https://docs.craftycontrol.com/pages/getting-started/installation/docker/)

[podman-systemd.unit - systemd units using Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
