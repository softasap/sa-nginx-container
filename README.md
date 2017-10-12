sa-nginx-container
==================

[![Build Status](https://travis-ci.org/softasap/sa-nginx-container.svg?branch=master)](https://travis-ci.org/softasap/sa-nginx-container)
[![License: MIT][softasap-license-image] ][softasap-license-url]
[![Ansible-Container friendly][ansible-container-image] ][ansible-container-url]

[ansible-container-image]: https://img.shields.io/badge/ansible--container-ready-brightgreen.svg
[ansible-container-url]: http://bit.ly/ansible-container
[softasap-license-image]: https://img.shields.io/badge/License-MIT-yellow.svg
[softasap-license-url]: https://opensource.org/licenses/MIT


Helper role to be executed with `ansible-container` aiming to build nginx based service for your application. Role is based on `sa-nginx` role,
your docker image might be any of ubuntu (14.04 LTS / 16.04 LTS), CentOS 7+, Fedora 25+, Alpine (3.4. 3.5 +)


Variables
---------

User inside container used to run
```
container_user: nginx
```

Container's nginx user id
```
container_uid: 1000
```

Container init system to setup: default dumb-init, also (will be)
supported phusion/base-image (based on ubuntu 16.04)
```
container_init: "dumb-init" # to support phusion at later stages

dumb_init_version: "1.2.0"
```

Directory to place nginx pid
```
nginx_pid_dir: /run/nginx
```

Interoperability with other containers - where to place static content,
and which directories need to be rsynced inside container.
```
nginx_static_dir: /static

nginx_asset_dirs: [] # directories to be copied inside docker container
```

In case if you don't provide your own nginx conf, container will use `sa-nginx`
scheme: site configs to be placed to `/etc/nginx/sites-enabled`, default nginx.conf
might be adjusted by specifying list of corrections in `nginx_conf_properties`

```
nginx_conf_properties:
  - {
      regexp: "^daemon *",
      line: "daemon off;",
      insertbefore: "BOF"
    }
  - {
      regexp: "^worker_processes *",
      line: "worker_processes auto;",
      insertbefore: "BOF"
    }
  - {
      regexp: "^pid *",
      line: "pid {{nginx_pid_dir}}/nginx.pid;",
      insertbefore: "BOF"
    }
```

Code in action
--------------

container.yml

we install nginx container role, and configure our application by applying some project
specific role in order to provide needed nginx tuning.


```

version: "2"
settings:
  conductor_base: ubuntu:16.04
  volumes:
   - temp-space:/tmp   # Used to copy static content between containers

services:
   www:
     from: ubuntu:16.04
     container_name: www
     roles:
       - {
           role: "softasap.sa-nginx-container"
         }
       - {
           role: "../standalone-fallback"
         }

volumes:
  temp-space:
    docker: {}

```

See box-example for the standalone working example. It will configure application
image that will display 'OK' on connect - check it out:

[![](https://github.com/play-with-docker/stacks/raw/cff22438cb4195ace27f9b15784bbb497047afa7/assets/images/button.png)](http://play-with-docker.com?stack=https://raw.githubusercontent.com/softasap/sa-nginx-container/master/box-example/docker-compose-try.yml)


Temporary hints
---------------

(1)

at a moment ansible-container is highly under development. You might spot issues, that are fixed in develop branch only.

In that case you might need to install ansible-container from source, i.e.

```shell

git clone https://github.com/ansible/ansible-container.git
cd ansible-container
git checkout develop
pip install -e .[docker,openshift]
```

If for some reason install is messed (manual packages updates, removals, etc) - try pip install with `--ignore-installed` flag.

later, when issue fix is released - to uninstall package installed in that way from source:

At {virtualenv}/lib/python2.7/site-packages/ (if not using virtualenv then {system_dir, like /usr/local}/lib/python2.7/dist-packages/)

remove the egg file (e.g. ansible-container.egg-link) if there is any;

from file easy-install.pth, remove the corresponding line (it should be a path to the source directory or of an egg file).

(2)

When using box-example, pay attention to `container.yml`, in particular, `conductor_base` should be derived
from the same distribution as you're building your target containers with, check list of currently supported base systems:


(3)
  If your system services are derived from different OS base images, than ... ?


Copyright and license
---------------------

Code licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) or the [MIT License] (http://opensource.org/licenses/MIT).

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)
