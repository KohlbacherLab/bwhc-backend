ARG JAVA_VERSION=11

FROM openjdk:${JAVA_VERSION}-jre AS builder

ARG BWHC_APP
ARG BWHC_ZIP=$BWHC_APP.zip

COPY $BWHC_ZIP /opt/

WORKDIR /opt

RUN unzip $BWHC_ZIP && rm $BWHC_ZIP


FROM openjdk:${JAVA_VERSION}-jre


LABEL org.opencontainers.image.licenses=MIT
LABEL org.opencontainers.image.source=https://github.com/KohlbacherLab/bwHC-REST-API-Gateway
LABEL org.opencontainers.image.description="Image of bwHC node backend with DNPM-Broker connector"


ARG BWHC_DATA_DIR=/bwhc_data
ARG BWHC_CONFIG_DIR=/bwhc_config

ARG BWHC_APP=$BWHC_APP
ENV BWHC_APP=$BWHC_APP

ENV PID_FILE=/dev/null
ENV APPLICATION_SECRET="$(head -c 64 /dev/urandom | base64)"

ENV BWHC_USER_DB_DIR=$BWHC_DATA_DIR/user-db
ENV BWHC_DATA_ENTRY_DIR=$BWHC_DATA_DIR/data-entry
ENV BWHC_QUERY_DATA_DIR=$BWHC_DATA_DIR/query-data
ENV BWHC_HGNC_DIR=$BWHC_DATA_DIR/hgnc

ENV PLAY_CONFIG=$BWHC_CONFIG_DIR/production.conf
ENV LOGBACK_CONFIG=$BWHC_CONFIG_DIR/logback.xml
ENV CONNECTOR_CONFIG=$BWHC_CONFIG_DIR/bwhcConnectorConfig.xml

# Set to a positive int N to activate generation of N random mtbfiles upon app start
ENV N_RANDOM_FILES=-1

USER 999

COPY --from=builder /opt/$BWHC_APP /opt/$BWHC_APP

VOLUME $BWHC_DATA_DIR
VOLUME $BWHC_CONFIG_DIR

EXPOSE 9000

ENTRYPOINT /opt/$BWHC_APP/bin/bwhc-rest-api-gateway \
    -Dplay.http.secret.key="$APPLICATION_SECRET" \
    -Dpidfile.path=$PID_FILE \
    -Dbwhc.zpm.site="$ZPM_SITE" \
    -Dbwhc.data.entry.dir=$BWHC_DATA_ENTRY_DIR \
    -Dbwhc.query.data.dir=$BWHC_QUERY_DATA_DIR \
    -Dbwhc.query.data.generate=$N_RANDOM_FILES \
    -Dbwhc.user.data.dir=$BWHC_USER_DB_DIR \
    -Dbwhc.hgnc.dir=$BWHC_HGNC_DIR \
    -Dconfig.file=$PLAY_CONFIG \
    -Dlogger.file=$LOGBACK_CONFIG \
    -Dbwhc.connector.configFile=$CONNECTOR_CONFIG
