# Quadlets

Collection with some of my Quadlet configuration files, or modifications of configurations found on the internet.

Feel free to contribute if you wrote a better implementation, some things like podman secrets may not have been added yet.

## List of containers and pods

- Jellyfin [View ↗](./jellyfin)
- Nextcloud (Apache) [View ↗](./nextcloud-apache)
- Nextcloud (Caddy) [View ↗](./nextcloud-caddy)
- Portainer [View ↗](./portainer)
- Pterodactyl [View ↗](./pterodactyl)
- Technitium [View ↗](./technitium-dns)

### Unused/not updated containers and pods

- Crafty [View ↗](./old/crafty)
- WG-Easy [View ↗](./old/wg-easy)

## Installation instructions

1. The default quadlet container directory for a rootless system is `~/.config/containers/systemd/`. Place all files from the container/pod in that directory. When running a rootful container, place them in a corresponding [system-wide directory](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path), like `/etc/containers`

2. Place the environment (.env) file(s) in `~/.config/containers/environment/` and modify them to your needs. If you need to place them in another directory, modify the container file(s) to point there.

2. Look in the container directory's README file for additional configuration steps and follow them. 

3. Reload the systemd daemon with `systemctl --user daemon-reload`, or when running podman as root: `sudo systemctl daemon-reload`

4. Start the service by running `systemctl --user start container-name.service`. When there are multiple container files, run the one ending with `-pod`. When running podman as root, run `sudo systemctl start container-name.service` instead. If the services don't show up, try the steps in *Debugging Podman Quadlets*

5. If necessary, open the services' ports in your firewall. With `firewalld` (default on RHEL-based distributions) this is done with `firewall-cmd` using `--add-service=` or `--add-port=` with `--permanent`

## Debugging Podman Quadlets

### Systemd generation fails

When quadlets aren't showing up in systemctl when trying to run them, it's possible to show the errors by doing a dry run:

```sh
/usr/libexec/podman/quadlet -dryrun -user
```

Omit `-user` when parsing rootful services

### Container fails to start

With a user that has permissions to view logs, look at the `journald` logs:

```sh
journalctl -xef _SYSTEMD_USER_UNIT=container-name.service
```

With running containers as root, replace `_SYSTEMD_USER_UNIT` with `_SYSTEMD_UNIT`

### Runtime podman logs

View the podman runtime logs like below. If there's not enough information try the instructions above for more detailed logs

```sh
podman logs container_name
```

## Quadlet documentation

[podman-systemd.unit - Podman documentation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#podman-rootful-unit-search-path)

[Make systemd better for Podman with Quadlet](https://www.redhat.com/en/blog/quadlet-podman)

# License

You're free to use, share and redistribute any of these configuration files however you like. All documentation (markdown files) is licensed under the [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)