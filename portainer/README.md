# Additional configuration

Enable the podman socket service:

```sh
systemctl --user enable --now podman.sock
```

Publish port `9090/tcp` or route your reverse proxy to it.

> [!NOTE]  
> The docs say to let the container run with `--privileged`, but just disabling SELinux labels for it like in the container file (`SecurityLabelDisable`) is less permissive and works fine. 

# Sources

[Install Portainer CE with Podman on Linux | Portainer documentation](https://docs.portainer.io/start/install-ce/server/podman/linux)