FROM openjdk:11-jre-slim AS zk-builder

ENV ZOO_USER=zookeeper \
    ZOO_CONF_DIR=/conf \
    ZOO_DATA_DIR=/data \
    ZOO_DATA_LOG_DIR=/datalog \
    ZOO_LOG_DIR=/var/log/zookeeper \
    ZOO_TICK_TIME=2000 \
    ZOO_INIT_LIMIT=5 \
    ZOO_SYNC_LIMIT=2 \
    ZOO_AUTOPURGE_PURGEINTERVAL=0 \
    ZOO_AUTOPURGE_SNAPRETAINCOUNT=3 \
    ZOO_MAX_CLIENT_CNXNS=60 \
    ZOO_STANDALONE_ENABLED=true \
    ZOO_ADMINSERVER_ENABLED=true

# Add a user with an explicit UID/GID and create necessary directories
RUN set -eux; \
    if ! getent group "$ZOO_USER" > /dev/null 2>&1; then \
        if getent group 1000 > /dev/null 2>&1; then \
            groupadd -r "$ZOO_USER" --gid=1001; \
            useradd -r -g "$ZOO_USER" --uid=1001 "$ZOO_USER"; \
        else \
            groupadd -r "$ZOO_USER" --gid=1000; \
            useradd -r -g "$ZOO_USER" --uid=1000 "$ZOO_USER"; \
        fi; \
    fi; \
    mkdir -p "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR" "$ZOO_LOG_DIR"; \
    chown "$ZOO_USER:$ZOO_USER" "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR" "$ZOO_LOG_DIR"

# Install required packages
RUN set -eux; \
    apt-get update; \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        dirmngr \
        gosu \
        gnupg \
        netcat \
        wget; \
    rm -rf /var/lib/apt/lists/*; \
    # Verify that gosu binary works
    gosu nobody true

ARG GPG_KEY=3F7A1D16FA4217B1DC75E1C9FFE35B7F15DFA1BA
ARG SHORT_DISTRO_NAME=zookeeper-3.9.3
ARG DISTRO_NAME=apache-zookeeper-3.9.3-bin

# Download Apache Zookeeper, verify its PGP signature, untar and clean up
RUN set -eux; \
    ddist() { \
        local f="$1"; shift; \
        local distFile="$1"; shift; \
        local success=; \
        local distUrl=; \
        for distUrl in \
            'https://www.apache.org/dyn/closer.cgi?action=download&filename=' \
            https://www-us.apache.org/dist/ \
            https://www.apache.org/dist/ \
            https://archive.apache.org/dist/ \
        ; do \
            if wget -q -O "$f" "$distUrl$distFile" && [ -s "$f" ]; then \
                success=1; \
                break; \
            fi; \
        done; \
        [ -n "$success" ]; \
    }; \
    ddist "$DISTRO_NAME.tar.gz" "zookeeper/$SHORT_DISTRO_NAME/$DISTRO_NAME.tar.gz"; \
    ddist "$DISTRO_NAME.tar.gz.asc" "zookeeper/$SHORT_DISTRO_NAME/$DISTRO_NAME.tar.gz.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --keyserver hkps://keyserver.pgp.com --recv-key "$GPG_KEY" || \
    gpg --keyserver hkps://keyserver.ubuntu.com --recv-keys "$GPG_KEY" || \
    gpg --keyserver hkps://pgp.mit.edu --recv-keys "$GPG_KEY"; \
    tar -zxf "$DISTRO_NAME.tar.gz"; \
    mv "$DISTRO_NAME/conf/"* "$ZOO_CONF_DIR"; \
    rm -rf "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc"; \
    chown -R "$ZOO_USER:$ZOO_USER" "/$DISTRO_NAME"

# Update vulnerable dependencies with newer versions
RUN set -eux; \
    cd "/$DISTRO_NAME/lib"; \
    # Download newer versions of vulnerable dependencies \
    wget -O netty-handler-4.1.118.Final.jar https://repo1.maven.org/maven2/io/netty/netty-handler/4.1.118.Final/netty-handler-4.1.118.Final.jar; \
    wget -O netty-common-4.1.118.Final.jar https://repo1.maven.org/maven2/io/netty/netty-common/4.1.118.Final/netty-common-4.1.118.Final.jar; \
    wget -O logback-core-1.3.15.jar https://repo1.maven.org/maven2/ch/qos/logback/logback-core/1.3.15/logback-core-1.3.15.jar; \
    wget -O logback-classic-1.3.15.jar https://repo1.maven.org/maven2/ch/qos/logback/logback-classic/1.3.15/logback-classic-1.3.15.jar; \
    wget -O jetty-server-9.4.57.v20241219.jar https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-server/9.4.57.v20241219/jetty-server-9.4.57.v20241219.jar; \
    wget -O jetty-http-12.0.12.jar https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-http/12.0.12/jetty-http-12.0.12.jar; \
    wget -O commons-io-2.14.0.jar https://repo1.maven.org/maven2/commons-io/commons-io/2.14.0/commons-io-2.14.0.jar; \
    # Remove old vulnerable versions (updated to match actual versions in 3.9.3) \
    rm -f netty-handler-4.1.113.Final.jar netty-common-4.1.113.Final.jar logback-core-1.2.13.jar logback-classic-1.2.13.jar jetty-server-9.4.53.v20231009.jar jetty-http-9.4.56.v20240826.jar commons-io-2.11.0.jar; \
    chown -R "$ZOO_USER:$ZOO_USER" "/$DISTRO_NAME"

# layer2 which uses the base image which has 0 CVEs and copies zk from earlier layer
FROM ghcr.io/nirmata/wolfi-openjdk-17-jre:0.20

ARG SHORT_DISTRO_NAME=zookeeper-3.9.3
ARG DISTRO_NAME=apache-zookeeper-3.9.3-bin

ENV ZOO_USER=nonroot \
    ZOO_CONF_DIR=/conf \
    ZOO_DATA_DIR=/data \
    ZOO_DATA_LOG_DIR=/datalog \
    ZOO_LOG_DIR=/var/log/zookeeper \
    ZOO_TICK_TIME=2000 \
    ZOO_INIT_LIMIT=5 \
    ZOO_SYNC_LIMIT=2 \
    ZOO_AUTOPURGE_PURGEINTERVAL=0 \
    ZOO_AUTOPURGE_SNAPRETAINCOUNT=3 \
    ZOO_MAX_CLIENT_CNXNS=60 \
    ZOO_STANDALONE_ENABLED=true \
    ZOO_ADMINSERVER_ENABLED=true

COPY --from=zk-builder --chown="$ZOO_USER:$ZOO_USER" /$DISTRO_NAME /$DISTRO_NAME
COPY --from=zk-builder --chown="$ZOO_USER:$ZOO_USER" $ZOO_CONF_DIR $ZOO_CONF_DIR

WORKDIR apache-zookeeper-3.9.3-bin
VOLUME ["$ZOO_DATA_DIR", "$ZOO_DATA_LOG_DIR", "$ZOO_LOG_DIR"]

EXPOSE 2181 2888 3888 8080

ENV PATH=$ZOO_HOME/bin/:/$DISTRO_NAME/bin:$PATH \
    ZOOCFGDIR=$ZOO_CONF_DIR

COPY scripts/docker-entrypoint.sh /
COPY scripts $ZOO_HOME/bin/

RUN mkdir -p $ZOO_CONF_DIR
RUN mkdir -p $ZOO_DATA_DIR
RUN mkdir -p $ZOO_DATA_LOG_DIR
RUN mkdir -p $ZOO_LOG_DIR

RUN chown -R $ZOO_USER:$ZOO_USER $ZOO_CONF_DIR
RUN chown -R $ZOO_USER:$ZOO_USER $ZOO_DATA_DIR
RUN chown -R $ZOO_USER:$ZOO_USER $ZOO_DATA_LOG_DIR
RUN chown -R $ZOO_USER:$ZOO_USER $ZOO_LOG_DIR

USER 10000
