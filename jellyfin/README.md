# Additional configuration

Open the needed ports from the following table or (in the case of HTTP/HTTPS) add them to your reverse proxy. The ports marked optional are commented out in the container file.

| Port | Protocol | Description                |
|------|----------|----------------------------|
| 8096 | tcp      | HTTP site                  |
| 8920 | tcp      | HTTPS site (optional)      |
| 7359 | udp      | Local discovery (optional) |
| 1900 | udp      | DNLA discovery (optional)  |

# Sources

[linuxserver/jellyfin - Docker Image](https://hub.docker.com/r/linuxserver/jellyfin)

[Configuration | Jellyfin](https://jellyfin.org/docs/general/administration/configuration/)
