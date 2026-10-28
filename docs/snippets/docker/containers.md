# Ready-to-run containers

## Jenkins

```bash
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
```

## Portainer

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```

## Ubuntu (temporary shell)

```bash
docker run -i -t --rm ubuntu:latest /bin/bash
```

## MeTube

```bash
docker run -d -p 8081:8081 -v .:/downloads ghcr.io/alexta69/metube
```
