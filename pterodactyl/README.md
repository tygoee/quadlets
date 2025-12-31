# Additional configuration

First and modify the timezone in [pterodactyl.env](./pterodactyl.env).

You also need to enable the podman socket service:

```sh
systemctl --user enable --now podman.sock
```

## Creating passwords

You need to create passwords for MariaDB, Redis and a database salt. It's recommended to use a password generator like the one from [Bitwarden](https://bitwarden.com/password-generator/). Constraints:

- MariaDB user password (`pterodactyl_mysql`): Will be passed as an environment variable, avoid non-alphanumeric characters
- MariaDB root password (`pterodactyl_mysql_root`): Same as the MariaDB user password
- Database Hashids salt (`pterodactyl_hashids`): 20 characters, alphanumeric
- Redis password (`pterodactyl_redis`): Same as MariaDB, as it will be passed as a command-line argument

```sh
echo -n "password_here" | podman secret create pterodactyl_mysql -
echo -n "password_here" | podman secret create pterodactyl_mysql_root -
echo -n "password_here" | podman secret create pterodactyl_hashids -
echo -n "password_here" | podman secret create pterodactyl_redis -
```

## Open ports

Add your own IP address/domain name in `APP_URL` in [pterodactyl.env](./pterodactyl-hosts.env) and in `AddHost` in [pterodactyl.pod](./containers/pterodactyl.pod).

### With a domain name and HTTPS on a reverse proxy

This pod is best used with a reverse proxy like Caddy or Nginx Proxy Manager. Point these domains to the corresponding ports in your reverse proxy:

- panel.example.com to port 8000
- node.example.com to port 8080 (dont open this publicly)

Port 443 is passed to the container to allow the reverse proxy to communicate, and to resolve WebSocket addresses

### Without a domain name or reverse proxy (without HTTPS)

If you don't have a domain name, just open port 8000 to access the panel. 

## Making a symlink to the container

After starting the pod (to create the `pterodactyl-data` directory), you will need to run the following to create a symlink as a user with sudo/root access. This required to keep the default server file directory because of a quirk in pterodactyl:

```
sudo ln -s ~/.local/share/containers/storage/volumes/pterodactyl-data/_data /var/lib/pterodactyl
```

## Connecting Wings to Pterodactyl Panel

Wings will remain in a failed state while doing this, as there's no config file. Start Pterodactyl with `systemctl --user start pterodactyl-pod.service` and create a panel user (administrator) and log in:

```sh
podman exec -it pterodactyl-panel php artisan p:user:make
```

- Click the cog icon in the top right corner for the admin panel
- Go to locations and create a new a location for the node
- Go to nodes and create a new node. Make sure to choose the following options and modify the rest to your choice:
    - **With a domain name and HTTPS on a reverse proxy**
        - FQDN: *node.example.com* (use your own domain)
        - Behind Proxy: *Behind Proxy*
        - Daemon port: *443*
    - **Without a domain name or reverse proxy**
        - FQDN: *localhost*
        - Communicate Over SSL: *Use HTTP Connection*
        - Behind Proxy: *Not Behind Proxy* (default)
        - Daemon port: *8080* (default)
    - Daemon Server File Directory: You will probably want to keep the default option (this is only possible if you have made a symlink, explained earlier)

- Once created, go to the Configuation tab. Create the file `~/.local/share/containers/storage/volumes/pterodactyl-config/_data/config.yml`, copy the config in there and change the port to 8080

Now, (re)start Wings, after which it should start in a succeeded state:

```sh
systemctl --user restart pterodactyl-wings.service
```

In the Node's About tab under Information, you should now stop seeing loading icons and start to see information about the node. Before running servers you still need to change the log driver:

- In `config.yml`, under `docker` > `log_config`, change `type` to `json-file`

Restart Wings again. Now you can allocate some ports you need to use for your server(s) in the *Allocation* tab. Under *IP Address* enter `0.0.0.0` (or `::` for IPv6). Now you're ready to create a server. **When creating a server, make sure to set Enable OOM Killer to *true*, as disabling the OOM Killer fails.**

For logs about Wings, look at `podman logs pterodactyl-wings`. If a server is created and fails to run, search for the container in `podman ps --all` and get its logs with `podman logs container-uuid`, `container-uuid` being a long string of letters and numbers

---

If creating a server fails with an error about `io.weight` not existing, do the following:

- To get your user's current cgroups to that confirm `io` is not included, run:

```sh
cat /sys/fs/cgroup/$(cat /proc/self/cgroup | cut -d: -f3)/cgroup.controllers
```

- First make a directory in `/etc/systemd/system/user@<uid>.service.d/` (leave uid empty for all users). To get your user id, run `id -u` as the user running podman.

- Inside the created directory, create a file named `delegate.conf` containing a `[Service]` option with a `Delegate=` field containing your delegated cgroups and add `io`. Find more information [here](https://www.freedesktop.org/software/systemd/man/latest/systemd.resource-control.html#Delegate=).

- Reload and reexec system both as root and the affected user(s):

```sh
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
systemctl --user daemon-reexec
systemctl --user daemon-reload
```

Read more in the first answer of [this GitHub issue](https://github.com/containers/podman/discussions/13569).

## Adding a database (optional)

This is optional, unless your server requires a database

- Create a secret for its root password:

```sh
echo -n "password_here" | podman secret create pterodactyl_db_root -
```

- Move `mariadb.container` from `optional/` to `~/.config/containers/systemd/`

- Change `Network=` in `pterodacyl.pod` under `[Container]`:

```ini
Network=pasta:-T,443,-T,3310
```

- Add a database user (here the username `servers` is used). More info [here](https://pterodactyl.io/tutorials/mysql_setup.html#creating-a-database-host-for-nodes):

```sh
podman exec -it pterodactyl-db mariadb -u root -p
# Enter MariaDB root password
```

```sql
CREATE USER 'servers'@'%' IDENTIFIED BY 'somepassword';
GRANT ALL PRIVILEGES ON *.* TO 'servers'@'%' WITH GRANT OPTION;
```

- Enter `quit;` to exit.

- Now open the web admin panel, go to Databases and add one with these options:
    - Host: `pterodactyl-db`
    - Port: `3310`
    - Username: the MariaDB username. I used `servers`
    - Password: The MariaDB user password you just created.

# Sources

[I Built the PERFECT Game Server with Pterodactyl and Docker | Techno Tim](https://technotim.live/posts/pterodactyl-game-server/)

[Getting Started | Pterodactyl](https://pterodactyl.io/panel/1.0/getting_started.html)

[Installing Wings | Pterodactyl](https://pterodactyl.io/wings/1.0/installing.html)

[wings/docker-compose.example.yml at develop · pterodactyl/wings](https://github.com/pterodactyl/wings/blob/develop/docker-compose.example.yml)

[Delegate | systemd.resource-control](https://www.freedesktop.org/software/systemd/man/latest/systemd.resource-control.html#Delegate=)

[How can I enable cgroup controllers while running podman in gnome terminal? · containers/podman · Discussion #13569](https://github.com/containers/podman/discussions/13569)

[Setting up MySQL | Pterodactyl](https://pterodactyl.io/tutorials/mysql_setup.html#creating-a-database-host-for-nodes)