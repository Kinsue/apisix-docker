#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM debian:bullseye-slim

ARG APISIX_VERSION=3.15.0

COPY --chmod=755 ./install-brotli.sh /install-brotli.sh

RUN set -ex; \
    arch=$(dpkg --print-architecture); \
    apt update; \
    apt-get -y install --no-install-recommends wget gnupg ca-certificates curl;\
    codename=`grep -Po 'VERSION="[0-9]+ \(\K[^)]+' /etc/os-release`; \
    case "${arch}" in \
      amd64) \
        wget -O - https://repos.apiseven.com/pubkey.gpg | apt-key add - \
        && echo "deb https://repos.apiseven.com/packages/debian $codename main" | tee /etc/apt/sources.list.d/apisix.list \
        ;; \
      arm64) \
        wget -O - https://repos.apiseven.com/pubkey.gpg | apt-key add - \
        && echo "deb https://repos.apiseven.com/packages/arm64/debian $codename main" | tee /etc/apt/sources.list.d/apisix.list \
        ;; \
    esac; \
    apt update \
    && apt install -y apisix=${APISIX_VERSION}-0 \
    && apt-get purge -y --auto-remove \
    && rm /usr/local/openresty/bin/etcdctl \
    && openresty -V \
    && apisix version \
    && cd / && ./install-brotli.sh && rm -rf /install-brotli.sh \
    && groupadd --system --gid 636 apisix \
    && useradd --system --gid apisix --no-create-home --shell /usr/sbin/nologin --uid 636 apisix \
    && chown -R apisix:apisix /usr/local/apisix \
    && apt-get -y purge --auto-remove curl wget gnupg --allow-remove-essential

WORKDIR /usr/local/apisix

ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin


USER apisix

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /usr/local/apisix/logs/access.log \
    && ln -sf /dev/stderr /usr/local/apisix/logs/error.log

EXPOSE 9080 9443

COPY ./docker-entrypoint.sh /docker-entrypoint.sh
COPY ./check_standalone_config.sh /check_standalone_config.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["docker-start"]


STOPSIGNAL SIGQUIT




#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# FROM debian:bullseye-slim
# 
# ARG APISIX_VERSION=3.15.0
# 
# ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin
# 
# WORKDIR /usr/local/apisix
# 
# RUN set -ex; \
#     arch=$(dpkg --print-architecture); \
#     apt-get update; \
#     apt-get -y install --no-install-recommends wget gnupg ca-certificates curl cmake unzip; \
#     codename=`grep -Po 'VERSION="[0-9]+ \(\K[^)]+' /etc/os-release`; \
#     case "${arch}" in \
#       amd64) \
#         wget -O - https://repos.apiseven.com/pubkey.gpg | apt-key add - \
#         && echo "deb https://repos.apiseven.com/packages/debian $codename main" | tee /etc/apt/sources.list.d/apisix.list \
#         ;; \
#       arm64) \
#         wget -O - https://repos.apiseven.com/pubkey.gpg | apt-key add - \
#         && echo "deb https://repos.apiseven.com/packages/arm64/debian $codename main" | tee /etc/apt/sources.list.d/apisix.list \
#         ;; \
#     esac; \
#     apt-get update \
#     && apt-get install -y apisix=${APISIX_VERSION}-0 \
#     && rm /usr/local/openresty/bin/etcdctl \
#     # Install Brotli
#     && BROTLI_VERSION="1.1.0" \
#     && wget -q https://github.com/google/brotli/archive/refs/tags/v${BROTLI_VERSION}.zip \
#     && unzip v${BROTLI_VERSION}.zip \
#     && cd ./brotli-${BROTLI_VERSION} \
#     && mkdir build \
#     && cd build \
#     && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local/brotli .. \
#     && cmake --build . --config Release --target install \
#     && cd ../.. \
#     && if [ -d "/usr/local/brotli/lib64" ]; then echo /usr/local/brotli/lib64 > /etc/ld.so.conf.d/brotli.conf; else echo /usr/local/brotli/lib > /etc/ld.so.conf.d/brotli.conf; fi \
#     && ldconfig \
#     && ln -sf /usr/local/brotli/bin/brotli /usr/bin/brotli \
#     && rm -rf brotli-${BROTLI_VERSION} v${BROTLI_VERSION}.zip \
#     # Create user and correct permissions in the same layer to avoid duplication
#     && groupadd --system --gid 636 apisix \
#     && useradd --system --gid apisix --no-create-home --shell /usr/sbin/nologin --uid 636 apisix \
#     && chown -R apisix:apisix /usr/local/apisix \
#     # Forward logs
#     && ln -sf /dev/stdout /usr/local/apisix/logs/access.log \
#     && ln -sf /dev/stderr /usr/local/apisix/logs/error.log \
#     # Cleanup
#     && apt-get purge -y --auto-remove wget gnupg curl cmake unzip \
#     && rm -rf /var/lib/apt/lists/* \
#     # Validation
#     && openresty -V \
#     && apisix version
# 
# USER apisix
# 
# EXPOSE 9080 9443
# 
# COPY ./docker-entrypoint.sh /docker-entrypoint.sh
# COPY ./check_standalone_config.sh /check_standalone_config.sh
# 
# ENTRYPOINT ["/docker-entrypoint.sh"]
# 
# CMD ["docker-start"]
# 
# STOPSIGNAL SIGQUIT
# Test 
