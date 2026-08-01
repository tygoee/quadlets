# Additional configuration

> [!NOTE]  
> Rootless networking might be less performant and is can be a pain to configure for more advanced situations.

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

> [!CAUTION]  
> `iptables` is an unmaintained driver in RHEL at of the time of writing, meaning it won't get security updates and probably will be removed soon

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

> [!WARNING]
> This is clumsy and I've never gotten it to work properly, with issues like DNS running on the same host failing to respond

There two ways to do this. Your host network will be accessible at `10.0.2.2`. For more info look at [this guide](https://github.com/eriksjolund/podman-networking-docs#outbound-tcpudp-connections-to-the-hosts-localhost) about networks in rootless Podman:

- `pasta` (default for rootless podman):

```ini
Network=pasta:--map-host-loopback=10.0.2.2
```

- `slirp4netns`: (it might need to be installed)

```ini
Network=slirp4netns:allow_host_loopback=true
```

## Forward traffic

You need to open port `51820/udp` to let wireguard traffic through and use port `51820/tcp` to manage wg-easy, which can be forwarded or routed through your reverse proxy. 

# Sources

[Using WireGuard Easy with Podman · wg-easy/wg-easy Wiki](https://github.com/wg-easy/wg-easy/wiki/Using-WireGuard-Easy-with-Podman)

[eriksjolund/podman-networking-docs: rootless Podman networking documentation with examples](https://github.com/eriksjolund/podman-networking-docs)

[Man page of passt](https://passt.top/builds/latest/web/passt.1.html)
