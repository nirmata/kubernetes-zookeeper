FROM azul/zulu-openjdk:17.0.8.1-17.44.53-jre

ENV ZK_USER=zookeeper \
    ZK_HOME=/opt/zookeeper \
    ZK_DATA_DIR=/var/lib/zookeeper/data \
    ZK_DATA_LOG_DIR=/var/lib/zookeeper/log \
    ZK_LOG_DIR=/var/log/zookeeper \
    PATH=$PATH:/opt/zookeeper/bin

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
    apt-get -y dist-upgrade; \
    rm -rf /var/lib/apt/lists/*; \
# Verify that gosu binary works
    gosu nobody true

ARG GPG_KEY=DFF24FB8323ADAC90E3CF36F729E61230EA917E9
ARG SHORT_DISTRO_NAME=zookeeper-3.6.3
ARG DISTRO_NAME=apache-zookeeper-3.6.3-bin

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
    gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" || \
    gpg --keyserver keys.openpgp.org --recv-keys "$GPG_KEY" || \
    gpg --keyserver keyserver.ubuntu.com --recv-keys "$GPG_KEY"; \
    gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz"; \
    tar -xvzf "$DISTRO_NAME.tar.gz" -C /opt;  \
    rm -rf "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc"; \
	apt-get autoremove -y wget; \
	rm -rf /var/lib/apt/lists/*;

# Add a user with an explicit UID/GID and create necessary directories
RUN set -eux; \
    groupadd -r $ZK_USER --gid=1000; \
    useradd -r -g $ZK_USER --uid=1000 $ZK_USER; \
    mkdir -p "$ZK_DATA_LOG_DIR" "$ZK_DATA_DIR" "$ZK_LOG_DIR" /usr/share/zookeeper /tmp/zookeeper /usr/etc/; \
    chown -R "$ZK_USER:$ZK_USER" /opt/$DISTRO_NAME "$ZK_DATA_LOG_DIR" "$ZK_DATA_DIR" "$ZK_LOG_DIR" /tmp/zookeeper; \
    ln -s /opt/$DISTRO_NAME /opt/zookeeper; \
    ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper; \
	ln -s /opt/zookeeper/bin/* /usr/bin; \
	ln -s /opt/zookeeper/lib/* /usr/share/zookeeper;

EXPOSE 2181 2888 3888
WORKDIR /opt/zookeeper

#Copy configuration generator script to bin
COPY scripts $ZK_HOME/bin/
