# solareenlo/alpine-hugo

Docker image for hugo static page generator (https://gohugo.io)


## Environment Variables

* `HUGO_THEME`
* `HUGO_WATCH` (set to any value to enable watching)
* `HUGO_DESTINATION` (Path where hugo will render the site. By default `/output`)
* `HUGO_REFRESH_TIME` (in seconds, only applies if not watching, if not set, the container will build once and exit)
* `HUGO_BASEURL`


## Executing

    docker run --name "my-hugo" -P -v $(pwd):/src solareenlo/alpine-hugo

Or, more verbosely, and with a specified output mapping:

    docker run --name "my-hugo" --publish-all \
           --volume $(pwd):/src \
           --volume /tmp/hugo-build-output:/output \
           solareenlo/alpine-hugo

Find your container:

    docker ps | grep "my-hugo"
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                   NAMES
    ba00b5c238fc        solareenlo/alpine-hugo:latest   "/run.sh"              7 seconds ago       Up 6 seconds        1313/tcp      my-hugo


## Building The Image Yourself (optional)

    docker build -t solareenlo/alpine-hugo:latest .

The image is conveniently small at **about 20 MB** thanks to [alpine](http://gliderlabs.viewdocs.io/alpine-hugo):

    docker images | grep hugo
    solareenlo/alpine-hugo:0.18   latest              b2e7a8364baa        1 second ago      21.9 MB



## Creating a new tag

Create a new git branch, change the line `ENV HUGO_VERSION=0.18` in `Dockerfile` and wire it in the Docker Hub accordingly.


## docker-compose

Using this docker image together with nginx for serving static data.

`docker-compose.yml`

```yaml
hugo:
  image: solareenlo/alpine-hugo:latest
  volumes:
    - ./src/:/src
    - ./output/:/output
  environment:
    - HUGO_REFRESH_TIME=3600
    - HUGO_THEME=mytheme
    - HUGO_BASEURL=mydomain.com
  restart: always

web:
  image: solareenlo/nginx-static
  volumes:
    - ./output:/var/www
  environment:
    - VIRTUAL_HOST=mydomain.com
  ports:
    - 80
  restart: always
```

`VIRTUAL_HOST` is set for use with jwilder's `nginx-proxy`:

`docker-compose.yml`

```yaml
proxy:
  image: jwilder/nginx-proxy
  ports:
    - 80:80
    - 443:443
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock
    - vhost.d:/etc/nginx/vhost.d:ro
  restart: always
```


## Update Image on Hugo Update

```./update.sh 0.55.6```
