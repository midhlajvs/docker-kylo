# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM openjdk:8-jdk AS kylok8sBuild
LABEL maintainer="Apache NiFi <dev@nifi.apache.org>"
LABEL site="https://nifi.apache.org"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"
ENV MAVEN_OPTS="-Xms512M -Xmx1536M"
ENV MAVEN_SKIP_RC="true"

USER root

RUN apt-get update && \
    apt-get install -y maven \
                       rpm

RUN git clone https://github.com/teradata/kylo.git /opt/kylok8s && \
    cd /opt/kylok8s && \
    git checkout tags/v0.9.1.3 && \
    sed -i 's/http\:\/\/repo1\.maven\.org\/maven2/https\:\/\/repo1\.maven\.org\/maven2/' pom.xml && \
    mvn clean install -DskipTests

RUN cd /opt/kylok8s && \
    mkdir install/install-tar/target/kylo && \
    tar -C install/install-tar/target/kylo -xvf install/install-tar/target/kylo-*-dependencies.tar.gz && \
    mkdir -p /opt/nifi/data/lib/app && \
    cp -a install/install-tar/target/kylo/setup/nifi/. /opt/nifi/data/lib && \
    mv /opt/nifi/data/lib/kylo-spark-*.jar /opt/nifi/data/lib/app

##########

FROM nexquare/nifi-parent:1.7.1
LABEL maintainer="Midhlaj VS <midhlaj.vs@nexquare.io>"
LABEL site="https://nifi.apache.org/"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

COPY --from=kylok8sBuild /opt/nifi/data/lib /opt/nifi/data/lib

USER root

RUN mkdir -p /opt/nifi/activemq && \
    mkdir -p /opt/nifi/mysql && \
    mkdir -p /opt/nifi/drivers && \
    mkdir -p /opt/nifi/nifi-1.7.1/lib/app && \
    mkdir -p /opt/nifi/dump && \
    mkdir -p /opt/nifi/drivers && \
    mkdir -p /opt/fake_data && \
    mkdir -p /var/dropzone && \
    chown -R nifi:nifi /var/dropzone && \
    chown -R nifi:nifi /opt/fake_data && \
    ln -f -s /opt/nifi/nifi-1.7.1 /opt/nifi/current

# Spark
RUN cd /opt && wget -q https://archive.apache.org/dist/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.6.tgz && \
  tar -xzf spark-2.3.1-bin-hadoop2.6.tgz && \
  ln -s spark-2.3.1-bin-hadoop2.6 spark && \
  rm spark-2.3.1-bin-hadoop2.6.tgz && \
  mkdir -p /opt/spark/lib

ENV PATH=$PATH:/opt/spark/bin

# Kylo JARS + Deps
RUN /opt/nifi/data/lib/create-symbolic-links.sh /opt/nifi nifi nifi && \
    wget https://repo1.maven.org/maven2/org/apache/activemq/activemq-all/5.15.0/activemq-all-5.15.0.jar -P /opt/nifi/activemq/ && \
    wget https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/2.2.4/mariadb-java-client-2.2.4.jar -P /opt/nifi/drivers/ && \
    wget https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/1.5.7/mariadb-java-client-1.5.7.jar -P /opt/nifi/mysql/ && \
    wget https://repo1.maven.org/maven2/org/apache/hive/hcatalog/hive-hcatalog-core/2.3.2/hive-hcatalog-core-2.3.2.jar -P /opt/nifi/drivers/ && \
    wget https://jdbc.postgresql.org/download/postgresql-42.2.4.jar -P /opt/spark/jars/

# Impala JDBC driver
RUN wget -q https://downloads.cloudera.com/connectors/impala_jdbc_2.5.42.1062.zip && \
    unzip impala_jdbc_2.5.42.1062.zip && \
    unzip ClouderaImpalaJDBC-2.5.42.1062/ClouderaImpalaJDBC4_2.5.42.zip && \
    mkdir -p /opt/nifi/drivers/impala && \
    mv ClouderaImpalaJDBC4_2.5.42/* /opt/nifi/drivers/impala

# Web HTTP(s) & Socket Site-to-Site Ports
EXPOSE 8080 8443 10000

# Apply configuration and start NiFi
ENTRYPOINT      ["/docker-entrypoint.sh"]
CMD             ["bin/nifi.sh", "run"]
