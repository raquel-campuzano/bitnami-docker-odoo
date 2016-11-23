[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-odoo/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-odoo/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/odoo)](https://hub.docker.com/r/bitnami/odoo/)
# What is Odoo?

> Odoo is a suite of web based open source business apps. Odoo Apps can be used as stand-alone applications, but they also integrate seamlessly so you get a full-featured Open Source ERP when you install several Apps.

https://odoo.com/

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run Odoo with a Database Container

Running Odoo with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run Odoo. You can use the following docker compose template:

```yaml
version: '2'

services:
  postgresql:
    image: 'bitnami/postgresql:latest'
    volumes:
      - 'postgresql_data:/bitnami/postgresql'
  application:
    image: 'bitnami/odoo:latest'
    ports:
      - '80:8069'
      - '443:8071'
    volumes:
      - 'odoo_data:/bitnami/odoo'
    depends_on:
      - postgresql
volumes:
  postgresql_data:
    driver: local
  odoo_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create odoo-tier
  ```

2. Start a PostgreSQL database in the network generated:

  ```bash
  $ docker run -d --name postgresql --net=odoo-tier bitnami/postgresql
  ```

  *Note:* You need to give the container a name in order to Odoo to resolve the host

3. Run the Odoo container:

  ```bash
  $ docker run -d -p 80:8069 --name odoo --net=odoo-tier bitnami/odoo
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence of the Odoo deployment, the above examples define docker volumes namely `postgresql_data` and `odoo_data`. The Odoo application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:
```yaml
version: '2'

  postgresql:
    image: 'bitnami/postgresql:latest'
    volumes:
      - '/path/to/postgresql-persistence:/bitnami/postgresql'
  odoo:
    image: bitnami/odoo:latest
    depends_on:
      - postgresql
    ports:
      - 80:8069
    volumes:
      - '/path/to/odoo-persistence:/bitnami/odoo'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create odoo-tier
  ```

2. Create a PostgreSQL container with host volume:

  ```bash
  $ docker run -d --name postgresql \
    --net odoo-tier \
    --volume /path/to/postgresql-persistence:/bitnami/postgresql \
    bitnami/postgresql:latest
  ```

  *Note:* You need to give the container a name in order to Odoo to resolve the host

3. Create the Odoo container with hist volumes:

  ```bash
  $ docker run -d --name odoo -p 80:8069 \
    --net odoo-tier \
    --volume /path/to/odoo-persistence:/bitnami/odoo \
    bitnami/odoo:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of PostgreSQL and Odoo, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Odoo container. For the PostgreSQL upgrade see https://github.com/bitnami/bitnami-docker-postgresql/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/odoo:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop odoo`
 * For manual execution: `$ docker stop odoo`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the odoo folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm odoo`
 * For manual execution: `$ docker rm odoo`

5. Run the new image

 * For docker-compose: `$ docker-compose start odoo`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name odoo bitnami/odoo:latest`

# Configuration
## Environment variables
 When you start the odoo image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:
```yaml
application:
  image: bitnami/odoo:latest
  ports:
    - 80:8069
  environment:
    - ODOO_PASSWORD=my_password
  volumes:
    - 'odoo_data:/bitnami/odoo'
  depends_on:
    - postgresql
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -e ODOO_PASSWORD=my_password -p 80:8069 --name odoo -v /your/local/path/bitnami/odoo:/bitnami/odoo --network=odoo-tier bitnami/odoo
```

Available variables:
 - `ODOO_EMAIL`: Odoo application email. Default: **user@example.com**
 - `ODOO_PASSWORD`: Odoo application password. Default: **bitnami**
 - `POSTGRESQL_USER`: Root user for the PostgreSQL database. Default: **postgres**
 - `POSTGRESQL_PASSWORD`: Root password for the PostgreSQL.
 - `POSTGRESQL_HOST`: Hostname for PostgreSQL server. Default: **postgresql**
 - `POSTGRESQL_PORT`: Port used by PostgreSQL server. Default: **5432**

### SMTP Configuration

To configure Odoo to send email using SMTP you can set the following environment variables:
 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_TLS`: Use TLS encription with SMTP. Default **true**

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:
```yaml
  application:
    image: bitnami/odoo:latest
    ports:
      - 80:8069
    environment:
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
```

 * For manual execution:

```bash
 $ docker run -d -e SMTP_HOST=smtp.gmail.com -e SMTP_PORT=587 -e SMTP_USER=your_email@gmail.com -e SMTP_PASSWORD=your_password -p 80:8069 --name odoo -v /your/local/path/bitnami/odoo:/bitnami/odoo --network=odoo-tier bitnami/odoo
```

# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop odoo`
  * For manual execution: `$ docker stop odoo`

2. Copy the Odoo data folder in the host:

  ```bash
  $ docker cp /your/local/path/bitnami:/bitnami/odoo
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with Odoo data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-odoo/issues), or submit a
[pull request](https://github.com/bitnami/bitnami-docker-odoo/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-odoo/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright (c) 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
