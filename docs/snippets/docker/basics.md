# Docker basics

## Dockerfile

```dockerfile title="Dockerfile"
# syntax=docker/dockerfile:1

FROM node:lts-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

## Build and run

```bash
docker build -t getting-started .
docker run -d -p 127.0.0.1:3000:3000 getting-started
```

## Manage containers

```bash
docker ps
docker stop <the-container-id>
docker rm <the-container-id>

# Stop and remove
docker rm -f <the-container-id>
```

Remove **all** running containers:

```bash
sudo docker rm --force $(sudo docker ps -q)
```

## Publish to Docker Hub

```bash
docker login -u YOUR-USER-NAME
docker tag getting-started YOUR-USER-NAME/getting-started
docker push YOUR-USER-NAME/getting-started
```

Run your published image:

```bash
docker run -dp 0.0.0.0:3000:3000 YOUR-USER-NAME/getting-started
```

## Docker Compose

```bash
docker-compose up
docker-compose -f config_file.yml up
```
