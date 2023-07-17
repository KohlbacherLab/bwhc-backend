# bwHC Backend: Docker

Docker components for the bwHC Backend (adapted from [https://github.com/CCC-MF/docker-bwHealthCloud/blob/main/Backend.Dockerfile](https://github.com/CCC-MF/docker-bwHealthCloud/blob/main/Backend.Dockerfile)).

The image contains the [backend app](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/tree/master) as obtained from the "sbt dist" task.
It is usable 'as is'; site-specific customization is performed through environment variables and externally provided config files.


## Registry

Images are available on the [GitHub Docker Registry](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/pkgs/container/bwhc-backend)


## Usage

The following environment variables are required to run the image (see also the [manual](https://ibmi-intra.cs.uni-tuebingen.de/display/ZPM/bwHC+Prototype+Manual#bwHCPrototypeManual-BasicBackendApplicationConfiguration)):

* `ZPM_SITE`: Your site name (used for diaplay purposes)

The following environment variables have default settings but can be overriden as needed:

* `APPLICATION_SECRET`: A random String of min. 32 chars. Default: $(head -c 64 /dev/urandom | base64)
* `N_RANDOM_FILES`: Set to a positive integer to activate generation of N random MTBFiles. Default: -1

In addition, the following volumes in the image must be mounted from the host file system:

* `/bwhc_config`: Contains the backend's configuration files [production.conf](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/production.conf), [logback.xml](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/logback.xml) and bwhcConnectorConfig.xml (the latter depends on the used connector: the [Peer-to-peer connector](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/bwhcConnectorConfig.xml) or [DNPM-Broker connector](https://github.com/KohlbacherLab/bwHC-Query-Service/tree/master/bwhc_broker_connector#configuration))

* `/bwhc_data`: The directory for data persistence. By default, the backend application will attempt to create the following sub-directories, unless they already exist:
    * `data-entry` (to override, explicitly set env. variable [`BWHC_DATA_ENTRY_DIR`](https://github.com/KohlbacherLab/bwhc-backend/blob/main/docker/Dockerfile#L33))
    * `query-data` (to override, explicitly set env. variable [`BWHC_QUERY_DATA_DIR`](https://github.com/KohlbacherLab/bwhc-backend/blob/main/docker/Dockerfile#L33))
    * `user-db` (to override, explicitly set env. variable [`BWHC_USER_DB_DIR`](https://github.com/KohlbacherLab/bwhc-backend/blob/main/docker/Dockerfile#L33))
    * `hgnc` (to override, explicitly set env. variable [`BWHC_HGNC_DIR`](https://github.com/KohlbacherLab/bwhc-backend/blob/main/docker/Dockerfile#L33))

The user running the backend application must thus have corresponding permissions on the data directory, e.g. a dedicated system user to operate bwhc. See also the [manual](https://ibmi-intra.cs.uni-tuebingen.de/display/ZPM/bwHC+Prototype+Manual#bwHCPrototypeManual-LocalSiteandPersistence).

See below for how to override the executing user by explicitly passing the system UserID (and optionally GroupID) in plain Docker command and Docker-compose, respectively.

The backend application will run on the container's Port 9000.


### General information about using Docker

You are required to have Docker (and Docker compose) installed on your host/server. Please read the following information about installing Docker, if you have to install it first: https://docs.docker.com/engine/install/

For more information about Docker und Docker compose, please have a look at the documentations: https://docs.docker.com/reference/

### Running the container using plain Docker

To run the container you can use the following command:

```
docker run --rm --name bwhc-backend \
  -p 9000:9000 \
  -e ZPM_SITE=Tübungen \
  -v /PATH/TO/HOST/DIR/bwhc_config:/bwhc_config \
  -v /PATH/TO/HOST/DIR/bwhc_data:/bwhc_data \
  --user UserID[:GroupID] \
  ghcr.io/kohlbacherlab/bwhc-backend:1.0-snapshot-broker-connector
```

Please be aware to replace `PATH/TO/HOST/DIR` with correct directory on your server, and also set the UserId (and GroupId).

### Running the container using Docker compose

Create a Docker compose file named `docker-compose.yml`, containing all information used above:

```
services:
  app:
    image: ghcr.io/kohlbacherlab/bwhc-backend:1.0-snapshot-broker-connector

    user: UserID[:GroupID]

    ports:
      - 9000:9000

    environment:
      ZPM_SITE: Tübingen

    volumes:
      - /PATH/TO/HOST/DIR/bwhc_config:/bwhc_config
      - /PATH/TO/HOST/DIR/bwhc_data:/bwhc_data
```

Use the following command to start the Docker container in background or dismiss option `-d` to start the container in foreground with the ability to stop it using ctrl-c:

```
docker-compose up -d
```

If you are using newer Docker installations, you might want to use build in `compose` subcommand (without '-'):

```
docker compose up -d
```
