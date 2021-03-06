# Nginx Autoconf (works with Swarm)
A docker container for automatically create nginx configuration files from a template inspired by [docker-gen](https://github.com/jwilder/docker-gen).

Nginx autoconf listens to Docker socket and regenerate conf files everytime a container starts or stops. It reloads all nginx containers after changing conf files automatically.

## Usage
### with docker compose
Create a compose file and add `nginx` and `arefaslani/nginx-autoconf` services. The `nginx` container must share `/etc/nginx/conf.d` directory. Because nginx-autoconf writes config files there. `nginx-autoconf` service mounts `/var/run/docker.sock` file so that it can listen to docker events and also nginx's `/etc/nginx/conf.d` directory.

Last thing to do is adding app service and add `app.virtual_host` and `app.virtual_port` labels to it.

```yaml
version: "3.1"

volumes:
  nginx-data:

services:
  nginx-autoconf:
    image: arefaslani/nginx-autoconf
    volumes:
      - nginx-data:/etc/nginx/conf.d
      - /var/run/docker.sock:/var/run/docker.sock

  nginx:
    image: nginx
    ports:
      - 80:80
    volumes:
      - nginx-data:/etc/nginx/conf.d

  app:
    image: arefaslani/docker-sample-express
    ports:
      - 3000:3000
    labels:
      - app.virtual_host=sample-express.test
      - app.virtual_port=3000
```

Nginx Autoconf will create a nginx conf file from the template and palces it in `/etc/nginx/conf.d` directory.
```nginx
upstream backend {
  server 10.0.0.2:3000 # ip will be fetched automatically
}

server {
  server_name sample-express.test;
  proxy_buffering off;

  location / {
    proxy_pass http://backend;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # HTTP 1.1 support
    proxy_http_version 1.1;
    proxy_set_header Connection "";
  }
}
```

Add this line to your `/etc/hosts` file:
```ruby
# IP must be the IP of docker host.
127.0.0.1 sample-express.test
```
Then run it with `docker-compose up` or
deploy your stack to the swarm by running
```bash
docker stack deploy --compose-file docker-compose.yml mystack
```
Thats all!

## Todo
* Support custom templates for apps
* Show how to setup https configuration with `Letsencrupt`
