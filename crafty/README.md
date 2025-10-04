# Crafty container

Quadlet files for setting up Crafty controller for Minecraft servers

## Setup

1. Place [crafty.container](./crafty.container) in `~/.config/containers/systemd` (or the corresponding [system directory](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path) when running with root)

2. Reload the systemd daemon with `systemd --user daemon-reload`

3. Start the crafty service by running `systemd --user start crafty.service`

After adding the following ports to your firewall configuration, you can access the webserver on port `8443`. It will automatically start on every boot from now on.

| Port(s)     | Protocol | Description                                                         |
| ----------- | -------- | ------------------------------------------------------------------- |
| 8443        | tcp      | Crafty web panel                                                    |
| 8123        | tcp      | Dynmap (optional)                                                   |
| 19132       | udp      | Minecraft Bedrock Edition server                                    |
| 25500-25600 | tcp      | Minecraft Java Edition servers (only use 25565 for a single server) |

## Sources

[Docker - Crafty Documentation](https://docs.craftycontrol.com/pages/getting-started/installation/docker/)

[podman-systemd.unit - systemd units using Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)
