## CI&T PHP Docker image(s)

These Docker image(s) intends to be a containerized PHP/Apache solution for multiple purposes.

The source code is available under GPLv3 at Github in this [link]((https://github.com/ciandt-dev/docker-hub-php).

By utilizing Docker technologies, that already provides an easy way of spinning up new environments along with its dependecies. This image can speed up developers which different backgrounds and equipments to create quickly a new local environment allowing them to easily integrate in automated tests and deployment pipelines.

At this moment we have the following version(s).

### [Acquia](#acquia)

Our intent is to be a Docker container that mimics PHP/Apache running on Acquia environment with the same version of softwares, packages, modules and its underlying operating system.

Acquia publishes a table with its platform infrastructure information on the link: https://docs.acquia.com/cloud-platform/arch/tech-platform

These images will have the following name pattern: __acquia-*YYYY-MM-DD-php{Major version}*__ (For example: __acquia-2021-04-12-php7.4__)

#### [*Bundled software versions*](#software-versions)

These are the currently software versions bundled in the image(s) by tag.

* acquia
  * Ubuntu 16.04-LTS
  * Apache 2.4.18
  * PHP 7.4.15 (plus extensions)
    * APCu 5.1.19
    * Gnupg 1.4.0
    * HTTP 3.2.4
    * Igbinary 3.2.1
    * Imagick 3.4.4
    * Memcache 3.1.5
    * Oauth 2.0.7
    * Propro 2.1.0
    * Raphf 2.0.1
    * SSH2 1.2beta
    * Xdebug 3.0.4
    * YAML 2.2.1
  * Dumb-init 1.2.0
  * __Pre-loaded scripts for customization__
    * Composer 2.0.12
    * Drush 8.4.2
    * Grunt CLI 1.2.0
      * Compass 1.1.1
    * Node.js 0.10.37
    * Ruby 2.3.0
      * Bundler 1.13.6
      * Compass 1.0.3

__*Deprecated*__

* acquia-2020-02-02-php7.3
* acquia-2019-09-09-php7.2
* acquia-2017-01-13
* acquia-2017-01-11
* acquia-2016-12-28
* acquia-2016-12-16
* acquia-2016-11-30
* acquia-2016-11-25
* acquia-2016-11-14

* * *

## [Quick Start](#quickstart)

__*Download the image*__

```
docker pull ciandt/php:acquia-2021-04-12-php7.4
```

__*Run a container*__

```
docker run \
  --name myContainer \
  --detach \
  ciandt/php:acquia-2021-04-12-php7.4
```

__*Check running containers*__

```
docker ps --all
```

* * *

## [Running standalone](#running-standalone)

If you just need the container there is a snippet that can help running in standalone mode.

```
# define variables
DOCKER_IMAGE_VERSION="2021-04-12-php7.4" # Update according which version you need.
HOST_CODE_FOLDER=""${HOME}"/workspace/mySite"
HOST_FILES_FOLDER=""${HOME}"/workspace/myNFSstorage"
DOCKER_CONTAINER_NAME="myContainer"
DOCKER_IMAGE="ciandt/php:acquia-${DOCKER_IMAGE_VERSION}"

# run your container
docker run \
  --volume "${HOST_CODE_FOLDER}":/var/www/html \
  --volume "${HOST_FILES_FOLDER}":/nfs \
  --name "${DOCKER_CONTAINER_NAME}" \
  --detach \
  "${DOCKER_IMAGE}"
```

After run, you can inquiry Docker service and get the IP address of your newly running container named __myContainer__ by using the following command:

```
docker inspect --format '{{ .NetworkSettings.IPAddress }} myContainer'
```

Let's suppose that the returned IP address was __172.17.0.2__.
Just open a browser and try to access:

> http://172.17.0.2

Your website should be displayed perfectly.

* * *

## [Customizing](#customizing)

As intended, you can take advantage from this image to build your own and already configure everything that a project requires.

### [CI&T scripts](#scripts)

There are available scripts to help customization:

- install-composer
- install-drush
- install-grunt
- install-nodejs
- install-ruby

Scripts are composed by two parts;

- one executable file *script-name*__.sh__
- one variables file *script-name*__.env__

The *script-name* __.env__ contains the variables that *script-name* __.sh__ requires to perform its task.

All scripts are located inside folder __/root/ciandt__ and must be declared in the *__Makefile__*. Thus, it is easy to run any of them and have its dependency. Please check the section in app/scripts/*__Makefile__* of the correspondent script to see if there is any other dependency that needs to declare additional environment variables.

Just to give an quick example, you can create your own Docker image based on this one that already ships Drush installed as well. A Dockerfile performing it could be like:

```
# Insert here your target image
FROM ciandt/php:acquia-2021-04-12-php7.4

# installs required package
RUN apt-get update \
    && apt-get install \
                --no-install-recommends \
                --assume-yes \
                make

# defines Composer and Drush versions
ENV COMPOSER_VERSION 2.0.12
ENV DRUSH_VERSION 8.4.2

# installs Drush
RUN cd /root/ciandt \
    && make install-drush
```

Then you just need to build / run your new customized Docker image and you are ready to go.

* * *

## [Running in Docker-Compose](#running-docker-compose)

Since a project is not going to use solely this container, it may need a Docker-Compose file.

Just to exercise, follow an example of this running customized and also behind a __Nginx__ proxy.

Create a new folder and fill with these 3 files and respective folders;

#### [__*conf/php.local.env*__](#php-env)

```
## Nginx proxy configuration
# https://hub.docker.com/r/jwilder/nginx-proxy/
VIRTUAL_HOST=mySite.local
```

##### [__app/php/Dockerfile__](#dockerfile)

```
# Insert here your target image
FROM ciandt/php:acquia-2021-04-12-php7.4

# installs required package
RUN apt-get update \
    && apt-get install \
                --no-install-recommends \
                --assume-yes \
                make

# defines Composer and Drush versions
ENV COMPOSER_VERSION 2.0.12
ENV DRUSH_VERSION 8.4.2

# installs Drush
RUN cd /root/ciandt \
    && make install-drush
```

##### [__*docker-compose.yml*__](#docker-compose)

```
php:
  build: ./php
  container_name: php
  env_file: ../conf/php.local.env

nginx:
  image: jwilder/nginx-proxy:latest
  container_name: nginx
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock:ro
  ports:
    - "80:80"
    - "443:443"
```

Then just spin-up your Docker-Compose with the command:

```
docker-compose up -d
```

Inspect Nginx container IP address:

```
docker inspect \
        --format \
        "{{.NetworkSettings.Networks.bridge.IPAddress }}" \
        "nginx"
```

Use the IP address to update __hosts__ file. Let's suppose that was 172.17.0.2.

Then, add an entry to __/etc/hosts__.
> 172.17.0.2 php.local

And now, try to access in the browser
> http://php.local

Voilà!
Your project now have Nginx and PHP/Apache up and running.
\\o/

* * *

## [User Feedback](#user-feedback)

### [*Issues*](#issues)

If you have problems, bugs, issues with or questions about this, please reach us in [Github issues page](https://github.com/ciandt-dev/docker-hub-php/issues).

__Needless to say__, please do a little research before posting.

### [*Contributing*](#contributing)

We gladly invite you to contribute fixes, new features, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a GitHub issue, especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.

### [*Documentation*](#documentation)

There are __two parts__ of the documentation.

First, in the master branch, is this README.MD. It explains how this little scripts framework work and it is published on [Github page](https://github.com/ciandt-dev/docker-hub-php).

Second, in each image version there is an additional README.MD file that explains how to use that specific Docker image version itself. __*Latest version*__ is always the one seen on [Docker Hub page](https://hub.docker.com/r/ciandt/php).

We strongly encourage reading both!

* * *

Please feel free to drop a message in the comments section.

Happy coding, enjoy!!

"We develop people before we develop software" - Cesar Gon, CEO
