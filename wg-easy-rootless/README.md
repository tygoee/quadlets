# Additional configuration

> Note: Rootless networking might be less performant and is can be a pain to configure for more advanced configurations.

## Create password hash secret

Run this command to create a password hash:

```sh
podman run --rm -it ghcr.io/wg-easy/wg-easy wgpw 'password'
```

To store it in a Podman secret, copy the hash in here and run it:

```sh
echo -n 'password hash' | podman secret create wg_easy_hash -
```

## Enable kernel modules

For each of the modules listed below, run `lsmod | grep module_name`, to see if they are running (running modules return their module name). Also check if `wireguard` and `xt_MASQUERADE` are running.

If they aren't, you need to create a file `/etc/modules-load.d/iptables.conf` with the following content:

```
ip_tables
iptable_filter
iptable_nat
```

If `wireguard` or `xt_MASQUERADE` are also missing, you need to add them in `/etc/modules-load.d/` as well. Now all modules will be automatically starting each boot.

To start the modules now, run `sudo modprobe module_name` for all modules that were missing.

## Allowing Wireguard connect to the host (optional)

There two ways to do this. Your host network will be accessible at `10.0.2.2`. For more info look at [this guide](https://github.com/eriksjolund/podman-networking-docs#outbound-tcpudp-connections-to-the-hosts-localhost) about networks in rootless Podman:

- `pasta` (default for rootless podman):

```ini
Network=pasta:--map-host-loopback=10.0.2.2
```

- `slirp4netns`: (it might need to be installed)

```ini
Network=slirp4netns:allow_host_loopback=true
```

# Sources

https://github.com/wg-easy/wg-easy/wiki/Using-WireGuard-Easy-with-Podman

https://github.com/eriksjolund/podman-networking-docs

https://passt.top/builds/latest/web/passt.1.html