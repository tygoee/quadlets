# Additional configuration

First, add your UID and GID in [`pterodactyl-wings.container`](./containers/pterodactyl-wings.container) and modify the timezone.

## Creating MariaDB passwords

You need to create a MariaDB user password and a MariaDB root password:

```sh
echo -n "password_here" | podman secret create pterodactyl_mysql -
echo -n "password_here" | podman secret create pterodactyl_mysql_root -
```

## Open ports

This is intended to be used with a reverse proxy. Open these ports. This will be needed to connect Panel to Wings. Change example.com for your own hostname, both here and in [pterodactyl.pod](./containers/pterodactyl.pod):

- panel.example.com to port 8000
- node.example.com to port 8080

Port 443 is passed to the container to allow the reverse proxy to communicate, and to resolve WebSocket addresses

## Making a symlink to the container

As a user with sudo/root access you will need to run the following to create a symlink. The reason will be explained later:

```
sudo ln -s /home/<user>/.local/share/containers/storage/volumes/pterodactyl-data/_data /var/lib/pterodactyl
```

## Connecting Wings to Pterodactyl Panel

Start Pterodactyl with `systemctl --user start pterodactyl-pod.service` and create a panel user (administrator) and log in:

```sh
podman exec -it pterodactyl-panel php artisan p:user:make
```

- Click the cog icon in the top right corner for the admin panel
- Go to locations and create a new a location for the node
- Go to nodes and create a new node. Make sure to choose the following options and modify the rest to your choice:
    - Behind Proxy: *Behind Proxy*
    - Daemon Server File Directory: You probably will want to keep the default option. This does require symlinking the directory to the container, because the podman socket tries to create that directory (only when creating a new server).
    - Daemon port: *443*
- Once created, go to the Configuation tab. Create the file `~/.local/share/containers/storage/volumes/pterodactyl-config/_data/config.yml`, copy the config in there and change the port to 8080

Now, (re)start Wings:

```sh
systemctl --user restart pterodactyl-wings.service
```

In the Node's About tab under Information, you should now stop seeing loading icons and start to see information about the node. For running servers you still need to change a few things:

- In `config.yml`, under `docker` > `log_config`, change `type` to `json-file`

Restart Wings again. When creating a server, make sure to set Enable OOM Killer to *true*, as disabling the OOM Killer fails.

For logs about Wings, look at `podman logs pterodactyl-wings`. If a server is created and fails to run, search for the container in `podman ps --all` and get its logs with `podman logs container-uuid`, `container-uuid` being a long string of letters and numbers

# Sources

<https://technotim.live/posts/pterodactyl-game-server/>

<https://pterodactyl.io/panel/1.0/getting_started.html>

<https://pterodactyl.io/wings/1.0/installing.html>

<https://github.com/pterodactyl/wings/blob/develop/docker-compose.example.yml>