# What is Liferay Portal?

**Liferay Portal** is an open source portal framework for building web applications, websites, and portals. It also offers an integrated CMS and may serve as an enterprise integration platform.  

[https://www.liferay.com/downloads-community](https://www.liferay.com/downloads-community)

![logo](https://github.com/igor-baiborodine/docker-liferay-portal-ce/blob/master/readme/logo.png?raw=true)
 
Logo &copy; Liferay, Inc.

# How to use this image

## Start a `liferay-portal` instance

```console
$ docker run --name <container name> -d %%IMAGE%%:<tag>
```

... where `<container name>` is the name you want to assign to your container and `<tag>` is the tag specifying the Liferay Portal CE version you want. See the list above for relevant tags.

The default Liferay Portal configuration is applied which features an embedded database (H2), and an embedded Elasticsearch instance. Please note that this setup is not suitable for production.

You can test it by visiting `http://container-ip:8080` in a browser. To get the container IP address, you can execute the following command:
```console
$ docker exec <container name> -it ifconfig
```
... or, if you need access outside the host, on port 80:

```console
$ docker run --name <container name> -p 80:8080 -d %%IMAGE%%:<tag>
```
You can then go to `http://localhost:80` or `http://host-ip:80` in a browser.

If you want to start a `liferay-instance` instance in debug mode, execute the following:
```console
$ docker run --name <container name> -d %%IMAGE%%:<tag> catalina.sh jpda run
```

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `liferay-portal`:

```yaml
version: '3'

services:
  liferay:
    image: %%IMAGE%%
    restart: always
    environment:
      LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED: "false"
      LIFERAY_TERMS_PERIOD_OF_PERIOD_USE_PERIOD_REQUIRED: "false"
      LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_ENABLED: "false"
      LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_CUSTOM_PERIOD_QUESTION_PERIOD_ENABLED: "false"
    ports:
      - "80:8080"
```

Run `docker-compose -f docker-compose.yml up`, wait for it to initialize completely, and visit `http://localhost:80` or `http://host-ip:80` (as appropriate).

## Check the Tomcat version information
```console
$ docker run --rm -it %%IMAGE%%:<tag> version.sh | grep 'Server version' 
``` 

## Container shell access and viewing Liferay Portal logs
The `docker exec` command allows you to run commands inside a Docker container. The following command line will give you a bash shell inside your `liferay-portal` container:
```console
$ docker exec -it <container name> bash
```

The Liferay Portal log is available through Docker's container log:
```console
docker logs -f <container name>
```

## Configure Liferay Portal via environment variables
You can override [portal.properties](https://github.com/liferay/liferay-portal/blob/master/portal-impl/src/portal.properties) by specifying corresponding environment variables, e.g.:
```properties
#
# Set this property to true if the Setup Wizard should be displayed the
# first time the portal is started.
#
# Env: LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED
#
setup.wizard.enabled=true
```
To override `setup.wizard.enabled` property, set `LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED` to `false` when running a new container: 
```console
$ docker run --name <container name> -p 80:8080 -it \ 
    --env LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED=false %%IMAGE%%:<tag>
```
Also, the environment variables can be set via docker-compose.yml (see example above) or by extending this image (see example below). 

## Healthcheck
This image does not contain an explicit health check. To add a healthcheck, you can start your `liferay-portal` instance with `--health-*` options:
```console
$ docker run --name <container name> -d \ 
    --health-cmd='curl -fsS "http://localhost:8080/c/portal/layout" || exit 1' \
    --health-start-period=1m \
    --health-interval=1m \
    --health-retries=3 \
    %%IMAGE%% 
```
... or by extending this image. For a more detailed explanation about why this image does not come with a default `HEALTHCHECK` defined, and for suggestions to implement your own health/liveness/readiness checks, you can read [here](https://github.com/docker-library/faq#healthcheck).

## Deploy modules to a running `liferay-portal` instance
This image exposes an optional `VOLUME` to allow deploying modules to a running container. To enable this option, you will need to:
1.	Create a deploy directory on a suitable volume on your host system, e.g. `/my/own/deploydir`.
2.	Start your `liferay-portal` instance like this:
```console
$ docker run --name <container name> -v /my/own/deploydir:/opt/liferay/deploy -d %%IMAGE%%:<tag>
```
The `-v /my/own/deploydir:/opt/liferay/deploy` part of the command mounts the `/my/own/deploydir` directory from the underlying host system as `/opt/liferay/deploy` inside the container to scan for layout templates, portlets, and themes to auto deploy.

## Where to store documents and media files
By default, Liferay Portal uses a document library store option called Simple File Store to store documents and media files on a file system (local or mounted). The store’s default root folder is `LIFERAY_HOME/data/document_library`.
There are several ways to store data used by applications that run in Docker containers. One of the options is to create a data directory on the host system (outside the container) and mount this to a directory visible from inside the container. This places the document and media files in a known location on the host system, and makes it easy for tools and applications on the host system to access the files. The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms on the host system are set up correctly.

You will need to:
1.	Create a data directory on a suitable volume on your host system, e.g. `/my/own/datadir`.
2.	Start your `liferay-portal` instance like this:
```console
$ docker run --name <container name> -v /my/own/datadir:/opt/liferay/data/document_library -d %%IMAGE%%:<tag>
```
The `-v /my/own/datadir:/opt/liferay/data/document_library` part of the command mounts the `/my/own/datadir` directory from the underlying host system as `/opt/liferay/data/document_library` inside the container, where Liferay Portal by default will store its documents and media files.

### Caveat
Do not use the default in-memory database(H2) when storing document and media files on the host system. You should configure your `liferay-portal` instance to use an external data source, e.g. MySQL.  

# How to Extend This Image

## Environment variables
If you would like to override the default configuration, i.e. portal.properties, you can do that by specifying corresponding environment variables in an image derived from this one:
```dockerfile
FROM %%IMAGE%%:<tag>

ENV LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED false
ENV LIFERAY_TERMS_PERIOD_OF_PERIOD_USE_PERIOD_REQUIRED false
ENV LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_ENABLED false
ENV LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_CUSTOM_PERIOD_QUESTION_PERIOD_ENABLED false
```

## Apply custom configuration to a `liferay-portal` instance

Volume LIFERAY_BASE
### Define portal-ext.properties
TODO

### Add new files or override existing ones
TODO 

### Execute custom shell scripts
TODO: LIFERAY_BASE/docker-entrypoint-initliferay.d

