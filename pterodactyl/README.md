# Additional configuration

> [!NOTE]  
> *Unfinished guide, podman-run argument [`--blkio-weight`](https://docs.podman.io/en/v4.6.1/markdown/options/blkio-weight.html) (automatically added by Pterodactyl) fails. Kernel arguments involing `cgroup2fs` don't seem to improve anything*

First, add your UID and GID in [`pterodactyl-wings.container`](./containers/pterodactyl-wings.container) and modify the timezone.

## Creating MariaDB passwords

You need to create a MariaDB user password and a MariaDB root password:

```sh
echo -n "password_here" | podman secret create pterodactyl_mysql -
echo -n "password_here" | podman secret create pterodactyl_mysql_root -
```

## Connecting Wings to Pterodactyl Panel

Start Pterodactyl with `systemctl --user start pterodactyl-pod.service` and create a panel user (administrator) and log in:

```sh
podman exec -it pterodactyl-panel php artisan p:user:make
```

- Click the cog icon in the top right corner for the admin panel
- Go to locations and create a new a location for the node
- Go to nodes and create a new node. Make sure to choose the following options and modify the rest to your choice (these first two options are for a reverse proxy that manages SSL certificates):
    - Communicate Over SSL: *Use HTTP Connection*  
    If this option is greyed out, right click the circle, click `Inspect`, double click the disabled attribute and remove it
    - Behind Proxy: *Behind Proxy*
    - Daemon Server File Directory: A directory on your host machine, that's owned by the user running Podman. If this is directory not owned by you, servers will fail to install.
- Once created, go to the Configuation tab. At the bottom, change `https://` to `http://` and then copy the config to a new file `~/.local/share/containers/storage/volumes/pterodactyl-config/_data/config.yml`.

Now, (re)start Wings:

```sh
systemctl --user restart pterodactyl-wings.service
```

In the Node's About tab under Information, you should now stop seeing loading icons and start to see information about the node. For running servers you still need to change a few things:

- In `config.yml`, under `docker` > `log_config`, change `type` to `journald`

When creating a server, make sure to set these options:

- Enable OOM Killer: true (disabling it fails)

# Sources

<https://technotim.live/posts/pterodactyl-game-server/>

<https://pterodactyl.io/panel/1.0/getting_started.html>

<https://pterodactyl.io/wings/1.0/installing.html>

<https://github.com/pterodactyl/wings/blob/develop/docker-compose.example.yml>