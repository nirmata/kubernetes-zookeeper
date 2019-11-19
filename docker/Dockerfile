FROM adoptopenjdk/openjdk12-openj9:x86_64-ubuntu-jre-12.0.2_10_openj9-0.15.1

ENV ZK_USER=zookeeper \
    ZK_HOME=/opt/zookeeper \
    ZK_DATA_DIR=/var/lib/zookeeper/data \
    ZK_DATA_LOG_DIR=/var/lib/zookeeper/log \
    ZK_LOG_DIR=/var/log/zookeeper \
    PATH=$PATH:/opt/zookeeper/bin

# Install required packges
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

ARG GPG_KEY=3F7A1D16FA4217B1DC75E1C9FFE35B7F15DFA1BA
ARG SHORT_DISTRO_NAME=zookeeper-3.5.5
ARG DISTRO_NAME=apache-zookeeper-3.5.5-bin

# Download Apache Zookeeper, verify its PGP signature, untar and clean up
RUN set -eux; \
    wget -q "https://www.apache.org/dist/zookeeper/$SHORT_DISTRO_NAME/$DISTRO_NAME.tar.gz"; \
    wget -q "https://www.apache.org/dist/zookeeper/$SHORT_DISTRO_NAME/$DISTRO_NAME.tar.gz.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" || \
    gpg --keyserver pgp.mit.edu --recv-keys "$GPG_KEY" || \
    gpg --keyserver keyserver.pgp.com --recv-keys "$GPG_KEY"; \
    gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz"; \
    tar -xvzf "$DISTRO_NAME.tar.gz" -C /opt; \
    rm -rf "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc"; \
    ln -s /opt/$DISTRO_NAME /opt/zookeeper; \
    rm -rf /opt/zookeeper/CHANGES.txt \
      /opt/zookeeper/README.txt \
      /opt/zookeeper/NOTICE.txt \
      /opt/zookeeper/CHANGES.txt \
      /opt/zookeeper/README_packaging.txt \
      /opt/zookeeper/build.xml \
      /opt/zookeeper/config \
      /opt/zookeeper/contrib \
      /opt/zookeeper/dist-maven \
      /opt/zookeeper/docs \
      /opt/zookeeper/ivy.xml \
      /opt/zookeeper/ivysettings.xml \
      /opt/zookeeper/recipes \
      /opt/zookeeper/src \
      /opt/zookeeper/$SHORT_DISTRO_NAME.jar.asc \
      /opt/zookeeper/$SHORT_DISTRO_NAME.jar.md5 \
      /opt/zookeeper/$SHORT_DISTRO_NAME.jar.sha1; \
	apt-get autoremove -y wget; \
	rm -rf /var/lib/apt/lists/*; \
	useradd $ZK_USER; \
    [ `id -g $ZK_USER` -eq 1000 ]; \
    mkdir -p $ZK_DATA_DIR $ZK_DATA_LOG_DIR $ZK_LOG_DIR /usr/share/zookeeper /tmp/zookeeper /usr/etc/; \
	chown -R "$ZK_USER:$ZK_USER" /opt/$DISTRO_NAME $ZK_DATA_DIR $ZK_LOG_DIR $ZK_DATA_LOG_DIR /tmp/zookeeper; \
	ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper; \
	ln -s /opt/zookeeper/bin/* /usr/bin; \
	ln -s /opt/zookeeper/lib/* /usr/share/zookeeper;

EXPOSE 2181 2888 3888
WORKDIR /opt/zookeeper

#Copy configuration generator script to bin
COPY scripts $ZK_HOME/bin/
