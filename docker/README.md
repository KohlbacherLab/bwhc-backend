# bwHC Backend: Docker 

Docker components for the bwHC Backend (adapted from [https://github.com/CCC-MF/docker-bwHealthCloud/blob/main/Backend.Dockerfile](https://github.com/CCC-MF/docker-bwHealthCloud/blob/main/Backend.Dockerfile)).

The image contains the [backend app](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/tree/master) as obtained from the "sbt dist" task.
It is usable 'as is'; site-specific customization is performed through environment variables and externally provided config files.


## Registry

Images are available at [https://github.com/orgs/KohlbacherLab/packages](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/pkgs/container/bwhc-backend)


## Usage

The following environment variables are required to run the image (see also the [manual](https://ibmi-intra.cs.uni-tuebingen.de/display/ZPM/bwHC+Prototype+Manual#bwHCPrototypeManual-BasicBackendApplicationConfiguration)):

* `ZPM_SITE`: Your site name (used for diaplay purposes)

The following environment variables have default settings but can be overriden as needed:

* `APPLICATION_SECRET`: A random String of min. 32 chars. Default: $(head -c 64 /dev/urandom | base64)
* `N_RANDOM_FILES`: Set to a positive integer to activate generation of N random MTBFiles. Default: -1

In addition, the following volumes in the image must be mounted from the host file system:

* `/bwhc_config`: Contains the backend's configuration files [production.conf](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/production.conf), [logback.xml](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/logback.xml) and bwhcConnectorConfig.xml (the latter depends on the used connector: the [Peer-to-peer connector](https://github.com/KohlbacherLab/bwHC-REST-API-Gateway/blob/master/deployment/bwhcConnectorConfig.xml) or [DNPM-Broker connector](https://github.com/KohlbacherLab/bwHC-Query-Service/tree/master/bwhc_broker_connector#configuration))
* `/bwhc_data`: The directory for data persistence


The backend application will run on the container's Port 9000. 


For instance, using docker compose:

```
services:
  app:
    image: bwhc-backend

    ports:
      - 9000:9000

    environment:
      ZPM_SITE: Tübingen

    volumes:
      - /PATH/TO/HOST/DIR/bwhc_config:/bwhc_config
      - /PATH/TO/HOST/DIR/bwhc_data:/bwhc_data
```


