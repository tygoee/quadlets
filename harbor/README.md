# Additional configuration

- Get the latest version number from the [Github repo](https://github.com/goharbor/harbor/releases). Edit all container files to include this version number (don't forget prepending the v). In `harbor-core.container`, also include the hostname.

- Download the offline installer and install the docker images. Replace HARBOR_VERSION with your version (again, don't forget prepending the v)

```sh
HARBOR_VERSION=v2.15.2
sudo dnf install -y wget tar
wget https://github.com/goharbor/harbor/releases/download/${HARBOR_VERSION}/harbor-offline-installer-${HARBOR_VERSION}.tgz
tar -xf harbor-offline-installer-${HARBOR_VERSION}.tgz
cd harbor
sudo podman load -i harbor.${HARBOR_VERSION}.tar.gz
```

You can verify the images are loaded by looking if they're listed in `sudo podman images`

- Copy the config directory from this repository, rename it to `harbor`, and move it to `/etc/harbor`. This is the folder that is normally installed with the prepare script, with passwords stripped and hostnames changed.

- Make it mountable by harbor containers:

```sh
sudo chown -R 10000:10000 /etc/harbor
```

- Create all these directories with the correct ownership:

```sh
sudo mkdir -p /var/lib/harbor/ca_download
sudo mkdir -p /var/lib/harbor/database
sudo mkdir -p /var/lib/harbor/job_logs
sudo mkdir -p /var/lib/harbor/redis
sudo mkdir -p /var/lib/harbor/registry
sudo mkdir -p /var/lib/harbor/trivy-adapter/reports
sudo mkdir -p /var/lib/harbor/trivy-adapter/trivy

sudo chown -R 10000:10000 /var/lib/harbor
sudo chown -R 999:999 /var/lib/harbor/database
sudo chown -R 999:999 /var/lib/harbor/redis
```


- Create podman secrets. Most secrets will be passed as environment variables. This means special characters can mess things up, so it's best to keep it to letters and numbers. When creating these files manually, don't forget to remove the trailing newline, which is automatically added by most editors!

```sh
openssl rand -base64 32 | head -c 32 > harbor_db
openssl rand -base64 32 | head -c 32 > harbor_registry
openssl rand -base64 32 | head -c 32 > harbor_csrf
openssl rand -base64 16 | head -c 16 > harbor_jobservice
openssl rand -base64 16 | head -c 16 > harbor_core
openssl rand -base64 16 | head -c 16 > harbor_secretkey
openssl rand -base64 8 | head -c 8 > harbor_robotscanner
openssl genrsa -traditional -out private_key.pem 4096

sudo podman secret create harbor_db harbor_db
sudo podman secret create harbor_registry harbor_registry
sudo podman secret create harbor_csrf harbor_csrf
sudo podman secret create harbor_jobservice harbor_jobservice
sudo podman secret create harbor_core harbor_core
sudo podman secret create harbor_secretkey harbor_secretkey
sudo podman secret create harbor_robotscanner harbor_robotscanner
sudo podman secret create harbor_key private_key.pem
```

- Add the `harbor_registry_user` user hash to the registry passwd file:

```sh
sudo dnf install -y httpd-tools # for htpasswd
cat harbor_registry | htpasswd -ciB /etc/harbor/registry/passwd harbor_registry_user
```
```

If you need to run HTTPS with own certificates from within the container rather than signing it via a reverse proxy, please look at the sources.

After this, you can start `harbor-core.service` to start harbor! You can point your reverse proxy to port 8080 and login with the initial username `admin` and password `Harbor12345`, make sure to change these. When startup fails and you're looking at the container logs one-by-one, it may help to roughly know in which order the containers are starting:

1. harbor-registry  
   harbor-registryctl  
   harbor-redis  
   harbor-db  
   harbor-portal
2. harbor-core  
   (harbor-trivy-adapter)
3. harbor-jobservice  
   harbor-exporter  
   harbor-nginx

For when using harbor as a proxy cache: when finished, create a registry and proxy cache project in harbor and add the registries to podman; this is an example `/etc/containers/registries.conf.d/100-harbor.conf` config:

```toml
[[registry]]
location = "docker.io"
blocked = true

[[registry.mirror]]
location = "reg.mydomain.com/docker-hub"

[[registry]]
location = "ghcr.io"
blocked = true

[[registry.mirror]]
location = "reg.mydomain.com/ghcr"
```

This will configure it for both rootful and rootless pulls. Verify this config by looking at `podman info`

# Sources

<https://github.com/goharbor/harbor/blob/main/make/photon/prepare/templates/docker_compose/docker-compose.yml.jinja>

<https://hackmd.io/@QI-AN/Install-Harbor-using-Podman> (translated)

[prepare.py - goharbor/harbor](https://github.com/goharbor/harbor/blob/main/make/photon/prepare/commands/prepare.py); this links to most files in [prepare/utils - goharbor/harbor](https://github.com/goharbor/harbor/tree/main/make/photon/prepare/utils)
