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

FROM apache/nifi:1.7.1
LABEL maintainer="Apache NiFi <dev@nifi.apache.org>"
LABEL site="https://nifi.apache.org"

ARG UID=1000
ARG GID=1000
ARG NIFI_VERSION=1.7.1
ARG MIRROR=https://archive.apache.org/dist

USER root

ENV NIFI_BASE_DIR /opt/nifi
ENV NIFI_TOOLKIT_HOME=$NIFI_BASE_DIR/nifi-toolkit-$NIFI_VERSION \
    NIFI_TOOLKIT_URL=/nifi/${NIFI_VERSION}/nifi-toolkit-${NIFI_VERSION}-bin.tar.gz

# Setup NiFi user
RUN mkdir -p $NIFI_BASE_DIR/data \
    && mkdir -p $NIFI_BASE_DIR/flowfile_repository \
    && mkdir -p $NIFI_BASE_DIR/content_repository \
    && mkdir -p $NIFI_BASE_DIR/provenance_repository \
    && mkdir -p $NIFI_BASE_DIR/logs \
    && mkdir -p $NIFI_BASE_DIR/data/templates \
    && chown -R nifi:nifi ${NIFI_BASE_DIR}

# Download, validate, and expand Nifi tools binary.
RUN curl -fSL ${MIRROR}/${NIFI_TOOLKIT_URL} -o ${NIFI_BASE_DIR}/nifi-toolkit-${NIFI_VERSION}-bin.tar.gz \
    && echo "$(curl ${MIRROR}/${NIFI_TOOLKIT_URL}.sha256) *${NIFI_BASE_DIR}/nifi-toolkit-${NIFI_VERSION}-bin.tar.gz" | sha256sum -c - \
    && tar -xvzf ${NIFI_BASE_DIR}/nifi-toolkit-${NIFI_VERSION}-bin.tar.gz -C ${NIFI_BASE_DIR} \
    && rm ${NIFI_BASE_DIR}/nifi-toolkit-${NIFI_VERSION}-bin.tar.gz \
    && chown -R nifi:nifi ${NIFI_HOME}

# Web HTTP(s) & Socket Site-to-Site Ports
EXPOSE 8080 8443 10000

COPY ./scripts/nifi-env.sh $NIFI_HOME/bin/nifi-env.sh
COPY ./scripts/docker-entrypoint.sh /
#COPY ./templates/ /${NIFI_BASE_DIR}/data/templates/
RUN chmod 700 /docker-entrypoint.sh

WORKDIR $NIFI_HOME

# Apply configuration and start NiFi
ENTRYPOINT      ["/docker-entrypoint.sh"]
CMD             ["bin/nifi.sh", "run"]
