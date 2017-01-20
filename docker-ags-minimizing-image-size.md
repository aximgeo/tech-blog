# ArcGIS for Server on Docker:  Minimizing Image Size

When creating docker images for ArcGIS Server (AGS) instances, keeping the image size down is a real challenge; while Linux-based images are small compared to using Windows instances, they're still quite large for a container.  The minimum required space for Linux AGS instances is between [3](http://server.arcgis.com/en/server/10.3/install/linux/arcgis-for-server-system-requirements.htm) to [10](http://server.arcgis.com/en/server/latest/install/linux/arcgis-for-server-system-requirements.htm) GB, depending on version:

```shell
$ docker images
ags10.3                    latest              8302e42c3065        20 hours ago        3.404 GB
ags10.4                    latest              44a2cf3be59c        7 days ago          5.71 GB
ags10.5                    latest              4a7cf2975201        5 days ago          6.838 GB
```

One way to minimize the overall image size is to keep the installer tarballs out of the images; the gzipped installers are a couple GB apiece:

```shell
$ ls -l --block-size=M
total 5526M
-rw-r--r-- 1 1011M Nov 15 06:48 ArcGIS_for_Server_Linux_1031_145870.tar.gz
-rw-r--r-- 1 2040M Oct 23 18:11 ArcGIS_for_Server_Linux_104_149446.tar.gz
-rw-r--r-- 1 2476M Nov 15 07:07 ArcGIS_Server_Linux_105_pr_153237.tar.gz
```

Since each command in the dockerfile will result in a separate commit to the image, this takes a bit of setup; one way to go about it is the following: 

* Make the installer tarballs accessible to the docker build process via a local webserver
* Inject the hostname and target install as arguments to the `docker build` process
* Download, extract, unzip, install and delete the tarball as a sincle docker `RUN` command

## Making the tarballs accessible
Download the tarball(s) from [ESRI](https://my.esri.com/#/downloads), and stage them on a network share or a local directory.  Once you have the installers in place, spin up a webserver with the root in your starging directory.  An easy way to accomplish this is to use the build-in Python webserver:

```shell
cd ags_linux_installers
$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

Once this is running, we can navigate to `localhost:8000` and see our installers there:

```shell
$ curl localhost:8000
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
<title>Directory listing for /</title>
<body>
<h2>Directory listing for /</h2>
<hr>
<ul>
<li><a href="ArcGIS_for_Server_Linux_1031_145870.tar.gz">ArcGIS_for_Server_Linux_1031_145870.tar.gz</a>
<li><a href="ArcGIS_for_Server_Linux_104_149446.tar.gz">ArcGIS_for_Server_Linux_104_149446.tar.gz</a>
<li><a href="ArcGIS_Server_Linux_105_pr_153237.tar.gz">ArcGIS_Server_Linux_105_pr_153237.tar.gz</a>
</ul>
<hr>
</body>
</html>
```

## Injecting the hostname as a build parameter
In order to download the tarball in our build process, we need to inject the hostname of the host machine into the build script; this can be done using the `$HOSTNAME` variable on Linux, or a combination of `%COMPUTERNAME%` and `%USERDNSDOMAIN%` on Windows.  We can pass this into the docker build process using the [build-arg](https://docs.docker.com/engine/reference/commandline/build/#/set-build-time-variables---build-arg) parameter:

```shell
docker build \
  --build-arg BUILD_HOST=$HOSTNAME:8000 \ # For windows, use $COMPUTERNAME.$USERDNSDOMAIN:8000
  --build-arg AGS_INSTALLER=ArcGIS_for_Server_Linux_104_149446.tar.gz
``` 

## Chaining commands to reduce commit layers
Now inside our dockerfile, we can download the tarball, unpack, install and delete it in a single commit by chaining commands together (using `&&`):

```shell
# Inject build variables
ARG BUILD_HOST
ARG AGS_INSTALLER

# Add license file
ADD arc.prvc /home/

WORKDIR /home

# Download the tarball with wget
RUN wget http://$BUILD_HOST/$AGS_INSTALLER && \

# Extract 
tar -xvzf $AGS_INSTALLER && \

# Delete the tarball
rm $AGS_INSTALLER && \ 

# Install
/home/ArcGISServer/Setup -m silent -l yes -a /home/arc.prvc -d /home/arcgis && \

# Delete the installation directory
rm -rf /home/ArcGISServer
```

And that's it--a simple way to shave a couple of gigs off of our AGS images.

You can find a full working sample of this script [here](https://gist.github.com/lobsteropteryx/372e59acf5aabd661011f8340ae99919).
