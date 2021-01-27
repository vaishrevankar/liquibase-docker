FROM oraclelinux:7

#RUN groupadd -g 1000 hunter
#RUN useradd -m -l -u 1000 -g hunter -s /bin/bash hunter
 
# Install GNUPG for package vefification and WGET for file download
RUN yum -y update \
    && yum -y install gnupg wget

RUN mkdir -p /tools/jdk1.8.0_271
RUN chown -R root:root /tools

WORKDIR /tools    
RUN chmod 777 /tools

COPY jdk1.8.0_271 /tools/jdk1.8.0_271
RUN chmod 777 /tools/jdk1.8.0_271
#RUN chown -R root:root /tools/jdk1.8.0_271

ENV JAVA_HOME /tools/jdk1.8.0_271
ENV PATH="$PATH:/tools/jdk1.8.0_271/bin"

# Make /liquibase directory and change owner to liquibase
RUN mkdir /liquibase && chown -R root:root /liquibase
WORKDIR /liquibase

# Latest Liquibase Release Version
ARG LIQUIBASE_VERSION=4.2.2

# Download, verify, extract
ARG LB_SHA256=807ef4b514d01fc62f7aaf4150a8435c90ccb5986f3272d3cfd1bd26c2cf7b4c
RUN set -x \
  && wget -O liquibase-${LIQUIBASE_VERSION}.tar.gz "https://github.com/liquibase/liquibase/releases/download/v${LIQUIBASE_VERSION}/liquibase-${LIQUIBASE_VERSION}.tar.gz" \
  && echo "$LB_SHA256  liquibase-${LIQUIBASE_VERSION}.tar.gz" | sha256sum -c - \
  && tar -xzf liquibase-${LIQUIBASE_VERSION}.tar.gz \
  && rm liquibase-${LIQUIBASE_VERSION}.tar.gz

# Setup GPG
RUN GNUPGHOME="$(mktemp -d)"
ENV PATH="$PATH:/liquibase"

# Copy Oracle JDBC libraries from local build folder
COPY mongo-java-driver-3.9.0-rc0.jar /liquibase/lib/.
COPY liquibase-mongodb-4.2.2.jar /liquibase/lib/.

COPY --chown=root:root docker-entrypoint.sh /liquibase
COPY --chown=root:root liquibase.docker.properties /liquibase
COPY --chown=root:root dbchangelog.xml /liquibase

RUN chmod 0755 /liquibase/docker-entrypoint.sh
RUN chmod 777 /liquibase

VOLUME /liquibase/classpath
VOLUME /liquibase/changelog

USER root
RUN chown -R root:root /liquibase

ENTRYPOINT ["/liquibase/docker-entrypoint.sh"]
CMD ["--help"]
