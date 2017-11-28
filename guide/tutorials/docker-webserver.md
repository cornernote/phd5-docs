Nginx
=====

Custom configuration `build/container-files/etc/nginx/conf.d/app.conf`.



### Customizing startup and webserver configuration

You can add your custom configuration to he container image on top of `dmstr/php-yii2` or `phd5-app`  
Grab the file from a running container by copying it into the `image-files` directory.

    mkdir -p image-files/etc/nginx/nginx.conf
    docker cp app_php_1:/etc/nginx/nginx.conf image-files/etc/nginx/nginx.conf
    
#### BASIC_AUTH example

Create a password file    
    
    htpasswd -c build/.htpasswd demo

Update Nginx server configuration

    location  /  {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
    
Add these updated configuration files to the build process    

    ADD build/.htpasswd /etc/nginx/.htpasswd
    ADD build/default /etc/nginx/sites-available/default

You can then `docker build` the image like described above.


#### Serve static files from PHP-fpm

Although it is strongly recommended to serve static files from nginx, there are use-cases when you need to generate
these files in on your PHP container.

To forward requests from nginx to PHP, just for `/assets`

    location /assets {
        # Fallback only
        try_files $uri $uri/ /asset.php?uri=$uri;
    }

On your PHP container create an `asset.php` file which serves your static files, make sure not to expose any sensitive
information.



Automated virtual hosts with Docker
-----------------------------------

### Via Docker networking

See `resources/reverse`, add the following to your application stack

```
services:
  php:
    [...]
    networks:
      - default
      - proxy
networks:
  proxy:
    external:
      name: reverse_default
```

### Via bridge device

To automatically create virtual hosts for your projects, you can use a combination of this [nginx-proxy](https://registry.hub.docker.com/u/jwilder/nginx-proxy/)
image and the [xip.io](http://xip.io) wildcard DNS service.

First, run the reverse-proxy container like described in its README, before you start web application containers.

```
docker pull jwilder/nginx-proxy
docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy
```

This will automatically setup virtual hosts accessible through port 80 on your Docker host.
You should now be able to access the application under 

Linux

- [myapp.127.0.0.1.xip.io](http://myapp.127.0.0.1.xip.io)
- [myapp.127.0.0.1.xip.io/admin](http://myapp.127.0.0.1.xip.io/backend)

OS X, Windows

- [myapp.192.168.59.103.xip.io](http://myapp.192.168.59.103.xip.io)
- [myapp.192.168.59.103.xip.io/admin](http://myapp.192.168.59.103.xip.io/admin)

> On OS X the command `echo $DOCKER_HOST` should print the IP of your host VM, replace it with `192.168.59.103` in `docker-compose.yml` and the URLs above, if neccessary.

You can display the application logs with:

```
fig logs
```

### Accessing from other clients on the network

If you need to access the application in development from another client (eg. mobile devices), you can setup a port forwarding to your host-vm. This is an example how to add port forwarding to VirtualBox VM.
 
```
boot2docker stop
VBoxManage modifyvm "boot2docker-vm" --natpf1 "rproxy,tcp,,8001,,80"
boot2docker start
```

Make sure to update your `VIRTUAL_HOST` environment variable in `docker-compose.yml`, replace `192.168.1.102` with the IP address of your machine.

```
VIRTUAL_HOST: myapp.127.0.0.1.xip.io,myapp.192.168.1.102.xip.io
```

and restart the containers with `docker-compose up -d web`.

You can access the application under the following URL

```
http://myapp.192.168.1.102.xip.io:8001
```

*More information on this topic available at [github.com/boot2docker](https://github.com/boot2docker/boot2docker/blob/master/doc/WORKAROUNDS.md).*
