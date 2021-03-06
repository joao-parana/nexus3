# nexus3

A Dockerfile for Sonatype Nexus Repository Manager 3, based on CentOS 7.

To run, binding the exposed port 8081 to the host.

```
docker run -d -p 8081:8081 --name nexus parana/nexus3
```

To test:

```
curl -u admin:admin123 http://localhost:8081/service/metrics/ping
```

To (re)build the image:

```
docker build -t parana/nexus3 .
```


## Notes

* Default credentials are: `admin` / `admin123`

* It can take some time (2-3 minutes) for the service to launch in a
new container.  You can tail the log to determine once Nexus is ready:

```
$ docker logs -f nexus
```

* Installation of Nexus is to `/opt/sonatype/nexus`.

* A persistent directory, `/nexus-data`, is used for configuration,
logs, and storage. This directory needs to be writable by the Nexus
process, which runs as UID 200.

* Three environment variables can be used to control the JVM arguments

  * `JAVA_MAX_MEM`, passed as -Xmx.  Defaults to `1200m`.

  * `JAVA_MIN_MEM`, passed as -Xms.  Defaults to `1200m`.

  * `EXTRA_JAVA_OPTS`.  Additional options can be passed to the JVM via
  this variable.

  These can be used supplied at runtime to control the JVM:

  ```
  docker run -d -p 8081:8081 --name nexus -e JAVA_MAX_HEAP=768m parana/nexus3
  ```


### Persistent Data

There are two general approaches to handling persistent storage requirements
with Docker. See [Managing Data in Containers](https://docs.docker.com/userguide/dockervolumes/)
for additional information.

  1. *Use a data volume container*.  Since data volumes are persistent
  until no containers use them, a container can created specifically for 
  this purpose.

  ```
  docker run -v /data --name nexus-data cogniteev/echo  \
         echo "data-only container for Nexus. To verify, execute: docker ps -a | nexus-data"
  docker logs nexus-data
  # you can save this container as a image using something like this:
  docker commit -a "João Antonio Ferreira <joao.parana@gmail.com>" \
         -m "Versão Inicial" \
         nexus-data \
         parana/nexus-data
  docker run -d -p 8081:8081 --name nexus --volumes-from nexus-data parana/nexus3
  ```

  2. *Mount a host directory as the volume*.  This is not portable, as it
  relies on the directory existing with correct permissions on the host.
  However it can be useful in certain situations where this volume needs
  to be assigned to certain specific underlying storage.

  ```
  mkdir -p nexus-data
  docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data parana/nexus3
  ```

### View the page 

Execute on host computer:

```
open http://localhost:8081
```