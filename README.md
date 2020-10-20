# Introduction

This repository is a fork of the [Robinyo/js-docker](https://github.com/Robinyo/js-docker) repository and has been 
updated to include support for building, configuring, and running **PostgreSQL** and **TIBCO JasperReports Server Community Edition 7.8** in containers.

Docker image for JasperServer Community Edition.  Please see [Jaspersoft Community](https://community.jaspersoft.com) for more information.

# Support
No warranty, explicit or implied, with usage of this repository.  As such, we don't offer any support for this project.

Please refer to the [Jaspersoft Forums](https://community.jaspersoft.com/questions) for support.

# Important!!

* JasperServer 7.8+ uses chrome/chromium for rendering, and phantomjs has been removed.  `scripts/installPackagesForJasperserver-ce.sh` takes care of installing all required packages.
* The keystore files [must be shared](https://community.jaspersoft.com/wiki/getting-invalid-credential-when-trying-login-tibco-jasperreports-server-75-two-nodes-server) ([archive](https://web.archive.org/web/20201020194425/https://community.jaspersoft.com/wiki/getting-invalid-credential-when-trying-login-tibco-jasperreports-server-75-two-nodes-server)) across all containers, including the `init` container.
* There are two dockerfiles for bulding CE images
  * `Dockerfile` - used for building the JasperServer application image
  * `Dockerfile-cmdline` - used for building the JasperServer init image.  The init image is used to create/init the database.
* `jasper.env` contains the details for connecting to a postgres instance.
* It appears that with 7.8 the `target` directory (`TIB_js-jrs-cp_7.8.0_bin.zip`/`jasperreports-server-cp-7.8.0-bin/target`) has been removed, and the libraries are now in the `TIB_js-jrs-cp_7.8.0_bin.zip`/`jasperreports-server-cp-7.8.0-bin/lib` directory.
* Version 7.8 fails with an error upon startup because of issue [TIBCOSoftware/jasperreports-server-ce#15](https://github.com/TIBCOSoftware/jasperreports-server-ce/issues/15).  The script `scripts/unpackWARInstaller-ce.sh` unzips the resource and `jasperserver.war` and properly copies `scripts/resfactory.properties` to `resources/jasperreports-server-cp-7.8.0-bin/jasperserver/WEB-INF/classes`.  See the referenced issue and [Community Question 822153](https://community.jaspersoft.com/questions/822153/error-closing-context) ([archive](https://web.archive.org/web/20170706181813/https://community.jaspersoft.com/questions/822153/error-closing-context)) for why that file is needed.
* The script `scripts/wait-for-container-to-exit.sh` from [Robinyo/js-docker](https://github.com/Robinyo/js-docker/blob/master/scripts/wait-for-container-to-exit.sh) effectively [polls](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) the `jasperreports-server-cmdline` container to determine when it completes (ie, database created and initialized).  Container names in Docker stacks [won't work](https://docs.docker.com/compose/compose-file/#container_name) since Docker creates dynamic names, and specifying the `container_name` attribute is just ignored.  This project changed that script to [inspect the image name](https://docs.docker.com/engine/api/v1.40/#operation/ImageList), **though the script remains untested**.  Below demonstrates the differences:

**Robinyo/js-docker** Way: 

`scripts/wait-for-container-to-exit.sh`:  
```
    INSPECT_CONTAINER=`curl --silent --unix-socket /var/run/docker.sock ${API_SERVER}/containers/${WAITFORIT_CONTAINER}/json`
```

`docker-compose.yml`:  
```yml
  jasperreports-server:
    command: ["/wait-for-container-to-exit.sh", "jasperreports-server-cmdline", "-t" , "30", "--", "/entrypoint-ce.sh", "run"]
  jasperreports-server-cmdline:
    container_name: jasperreports-server-cmdline
```


**This Project's** Way:

`scripts/wait-for-container-to-exit.sh`:  
```
    INSPECT_CONTAINER=`curl --silent --unix-socket /var/run/docker.sock ${API_SERVER}/v1.40/containers/json?filters=%7B%22ancestor%22%3A%5B%22${WAITFORIT_CONTAINER}%22%5D%7D`
```


`docker-compose.yml`:  
```yml
  jasperreports-server:
    command: ["/wait-for-container-to-exit.sh", "jasperserver-ce-init%3A7.8.0", "-t" , "30", "--", "/entrypoint-ce.sh", "run"]
  jasperserver-init:
    image: jasperserver-ce-init:7.8.0
```

In reality, though, because `scripts/wait-for-container-to-exit.sh` changes are untested, the `docker-compose.yml` looks like such:

```yml
  jasperreports-server:
    command: ["/entrypoint-ce.sh", "run"]
#     command: ["/wait-for-container-to-exit.sh", "jasperserver-ce-init%3A7.8.0", "-t" , "30", "--", "/entrypoint-ce.sh", "run"]
```

# Pre-built Docker image
Prebuilt docker images are available at [agois/jasperserver-ce:7.8](https://hub.docker.com/r/agois/jasperserver-ce/) and [agois/jasperserver-ce-init:7.8](https://hub.docker.com/r/agois/jasperserver-ce-init/).

# Building your own image
Download the latest JasperServer (tested with 7.8) and put in `./resources`: [JasperServer CE Releases](http://community.jaspersoft.com/project/jasperreports-server/releases)

Unpack and correct JasperServer:

```bash
cd scripts
chmod +x unpackWARInstaller-ce.sh 
./unpackWARInstaller-ce.sh 
```

Build a new Docker image and store it in local Docker (run from root of this project);

```bash
docker build -t jasperserver-ce:7.8.0 -f Dockerfile . 
docker build -t jasperserver-ce-init:7.8.0 -f Dockerfile-cmdline .
```

Tag the local Docker image with the Docker Hub username/repo:version
```bash
docker tag jasperserver-ce:7.8.0 agois/jasperserver-ce:7.8.0
```

Push new Docker image to Docker Hub
```bash
docker push agois/jasperserver-ce:7.8.0
```

# Adding fonts to Jasper Server
In `docker-compose.yml`, add the fonts jars in the section `jasperserver`/`volumes`:

```yml
    volumes:
      - ./jasper-fonts-1.0.0.jar:/usr/local/tomcat/webapps/jasperserver/WEB-INF/lib/jasper-fonts-1.0.0.jar
```

# Using with Docker Compose
You can use our [agois/jasperserver-ce](https://hub.docker.com/r/agois/jasperserver-ce/)/[agois/jasperserver-ce-init](https://hub.docker.com/r/agois/jasperserver-ce-init/) or your custom images.

Deploy local services using your `docker-compose.yml`.
```bash
docker stack deploy -c docker-compose.yml jasperserver-ce
```

Here is a sample `docker-compose.yml`:
```
version: '3.8'

services:

  postgres:    
    image: postgres:9.5.22
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - reportsnet
    env_file: jasper.env

  jasperserver:
    image: jasperserver-ce:7.8.0
    ports:
      - "11001:8080"
      - "11443:8443"
    depends_on:
      - jasperserver-init      
    env_file: jasper.env
    environment:
      - DB_HOST=postgres
    volumes:
      - jasper_webapp:/usr/local/tomcat/webapps/jasperserver
      - jasper_home:/usr/local/share/jasperserver
      - jasper_license:/usr/local/share/jasperserver/license 
      - jasper_keystore:/usr/local/share/jasperserver/keystore
      - jasper_customization:/usr/local/share/jasperserver/customization
    networks:
      - reportsnet
    command: ["/entrypoint-ce.sh", "run"]
#     command: ["/wait-for-container-to-exit.sh", "jasperserver-ce-init", "-t" , "30", "--", "/entrypoint-ce.sh", "run"]

  jasperserver-init:
    image: jasperserver-ce-init:7.8.0
    deploy:
      restart_policy:
        condition: none
    env_file: jasper.env
    volumes:
      - jasper_init_home:/usr/local/share/jasperserver
      - jasper_keystore:/usr/local/share/jasperserver/keystore
    environment:
      - DB_HOST=postgres
      - JRS_DBCONFIG_REGEN=true
    depends_on:
      - postgres
    command: ["/wait-for-it.sh", "postgres:5432", "-t" , "30", "--", "/entrypoint-cmdline-ce.sh", "init"]
    networks:
      - reportsnet

networks:
  reportsnet:

volumes:
  pgdata:
  jasper_home:
  jasper_webapp:
  jasper_license:
  jasper_keystore:
  jasper_customization:
  jasper_init_home:
```

*jasper.env*
```
DB_USER=postgres
DB_PASSWORD=postgres
DB_PORT=5432
DB_NAME=jasperserver
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```
