# Supported tags and respective `Dockerfile` links

-       [`5.0.2`, `5.0`, `5`, `latest` (*5.0.2/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/5.0/5.0.2/debian/Dockerfile)
-       [`4.3.7`, `4.3`, `4` (*4.3.7/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/4.3/4.3.7/debian/Dockerfile)
-       [`5.0.2-alpine`, `5.0-alpine`, `5-alpine` (*5.0.2/alpine/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/5.0/5.0.2/alpine/Dockerfile)
-       [`4.3.7-alpine`, `4.3-alpine`, `4-alpine` (*4.3.7/alpine/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/4.3/4.3.7/alpine/Dockerfile)

# What is Plone?

Plone is a free and open source content management system built on top of the Zope application server.

Plone is positioned as an "Enterprise CMS" and is most commonly used for intranets and as part of the web presence of large organizations. High-profile public sector users include the U.S. Federal Bureau of Investigation, Brazilian Government, United Nations, City of Bern (Switzerland), New South Wales Government (Australia), and European Environment Agency. Plone's proponents cite its security track record and its accessibility as reasons to choose Plone.

> [wikipedia.org/wiki/Plone_(software)](https://en.wikipedia.org/wiki/Plone_%28software%29)

# How to use this image

## Start a Plone instance

```console
$ docker run -p 8080:8080 plone/plone
```

This image includes `EXPOSE 8080` (the Plone port), so standard container linking will make it automatically available to the linked containers. Now you can add a Plone Site at http://localhost:8080 - default Zope user and password are `admin/admin`.

## Start Plone as a ZEO client

Considering ZEO server is running at `192.168.1.1` on default port `8100` you can do:

```console
$ docker run -e ZEO_ADDRESS=192.168.1.1:8100 -p 8080:8080 plone/plone
```

or, using the ZEO Docker Image:

```console
$ docker run --name=zeo plone/zeoserver
$ docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8080:8080 plone/plone
```

## Start Plone in debug mode

You can also start Plone in debug mode (`fg`) by running

```console
$ docker run -p 8080:8080 plone/plone fg
```

Still, this will not allow you to add `pdb` breakpoints. For this, you'll have to run Plone inside container like:

```console
$ docker run -it -p 8080:8080 plone/plone bash
  $ bin/instance fg
```

or with an already running container:

```console
$ docker exec -it my_running_plone bash
  $ bin/instance stop
  $ bin/instance fg
```

## Environment Variables

The Plone image uses several environment variable that allow to specify a more specific setup.

### `ZEO_ADDRESS`

This environment variable allows you to run Plone image as a ZEO client.

### `ZEO_READ_ONLY`

Defaults to `off`. Run Plone as a read-only ZEO client.

### `ZEO_CLIENT_READ_ONLY_FALLBACK`

Defaults to `false`. A flag indicating whether a read-only remote storage should be acceptable as a fallback when no writable storages are available.

### `ZEO_SHARED_BLOB_DIR`

Defaults to `off`. Set this to on if the ZEO server and the instance have access to the same directory.

### `ZEO_STORAGE`

Defaults to `1`. Set the storage number of the ZEO storage.

### `ZEO_CLIENT_CACHE_SIZE`

Defaults to `128MB`. Set the size of the ZEO client cache.

## How to extend this image

In order to run Plone with your custom theme or Plone Add-ons, you'll have to build another image based on this one. For this, you'll need to create two files, `site.cfg` which is a [zc.buildout](https://pypi.python.org/pypi/zc.buildout/2.5.0) configuration file, and [Dockerfile](https://docs.docker.com/engine/reference/builder/) which is the Docker recipe for your image.

### `site.cfg`

```console
[buildout]
extends = buildout.cfg
[instance]
eggs += plone.awesome.addon
```

### `Dockerfile`

```dockerfile
FROM plone/plone:5
COPY site.cfg /plone/instance/
RUN bin/buildout -c site.cfg
```

Build your custom Plone image

```console
$ docker build -t plone:custom .
```

Run it

```console
$ docker run -p 8080:8080 plone:custom
```

Test it at http://localhost:8080

## Where to Store Data

Important note: There are several ways to store data used by applications that run in Docker containers. We encourage users of the `plone` images to familiarize themselves with the options available.

The Docker documentation is a good starting point for understanding the different storage options and variations, and there are multiple blogs and forum postings that discuss and give advice in this area.

### Data-only containers (suitable for production use)

Let Docker manage the storage of your database data [by writing the database files to disk on the host system using its own internal volume management](https://docs.docker.com/engine/userguide/containers/dockervolumes/#creating-and-mounting-a-data-volume-container). The advantages of this approach is that you can deploy your Plone stack anywhere, without having to prepare hosts in advance or care about read/write permission or selinux policy rules. The downside is that the files may be hard to locate for tools and applications that run directly on the host system, i.e. outside containers.

-	Create the data container

	```console
	$ docker run --name plone_data \
	             -v /data/blobstorage \
	             -v /data/filestorage \
	         busybox chown -R 500:500 /data
	```

-	Use data container with Plone

	```console
	$ docker run --name plone_one \
	             --volumes-from plone_data \
	             -p 8080:8080 \
	         plone/plone
	```

Or with [Docker Compose](https://docs.docker.com/compose/)

-	Add docker-compose.yml file

	```console
	  plone:
	    image: plone/plone:5
	    volumes_from:
	    - plone_data
	    ports:
	    - "8080:8080"
	  plone_data:
	    image: busybox
	    volumes:
	    - /data/filestorage
	    - /data/blobstorage
	    command: ['chown', '-R', '500:500', '/data']
	```

-	Start Plone stack

	```console
	$ docker-compose up
	```

### Mount host directories as data volumes (suitable for development use)

Create data directories on the host system (outside the container) and [mount these to a directory visible from inside the container](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume). This places the database files in a known location on the host system, and makes it easy for tools and applications on the host system to access the files. The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms on the host system are set up correctly.

-	Create a data directories on a suitable volume on your host system, e.g. `/path/to/filestorage` and `/path/to/blobstorage`
-	Start your `plone` container like this:

	```console
	$ docker run -v /path/to/filestorage:/data/filestorage -v /path/to/blobstorage:/data/blobstorage -d plone/plone
	```

The `-v /path/to/filestorage:/data/filestorage` part of the command mounts the `-v /path/to/filestorage` directory from the underlying host system as `/data/filestorage` inside the container, where Plone will look for/create the `Data.fs` database file.

The `-v /path/to/blobstorage:/data/blobstorage` part of the command mounts the `-v /path/to/blobstorage` directory from the underlying host system as `/data/blobstorage` where blobs will be stored.

Make sure that Plone has access to read/write within these folders:

```console
$ chown -R 500:500 /path/to/filestorage /path/to/blobstorage
```

Note that users on host systems with SELinux enabled may see issues with this. The current workaround is to assign the relevant SELinux policy type to the new data directory so that the container will be allowed to access it:

```console
$ chcon -Rt svirt_sandbox_file_t /path/to/filestorage /path/to/blobstorage
```
