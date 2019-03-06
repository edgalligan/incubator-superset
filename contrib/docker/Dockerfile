#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM python:3.6

RUN useradd --user-group --create-home --no-log-init --shell /bin/bash superset

# Configure environment
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

RUN apt-get update -y

# Install dependencies to fix `curl https support error` and `elaying package configuration warning`
RUN apt-get install -y apt-transport-https apt-utils

# Install superset dependencies
# https://superset.incubator.apache.org/installation.html#os-dependencies
RUN apt-get install -y build-essential libssl-dev \
    libffi-dev python3-dev libsasl2-dev libldap2-dev libxi-dev

# Install extra useful tool for development
RUN apt-get install -y vim less postgresql-client redis-tools

RUN apt-get purge -y nodejs

RUN apt-get install ca-certificates

# Install nodejs for custom build
# https://superset.incubator.apache.org/installation.html#making-your-own-build
# https://nodejs.org/en/download/package-manager/

RUN curl -skfL https://deb.nodesource.com/setup_10.x | sed 's/curl -s/curl -sk/g' | sed 's/apt-get update/apt-get update -o Acquire::https::Verify-Peer=false/g' | bash -x -E - \
    &&  apt-get install -y nodejs -o Acquire::https::Verify-Peer=false

RUN nodejs -v && npm -v

WORKDIR /home/superset

COPY requirements.txt .
COPY requirements-dev.txt .

RUN pip install --trusted-host pypi.org pip --upgrade \ 
    && pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org --upgrade setuptools pip 

RUN pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org pytest-runner 

RUN pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org -r requirements.txt -r requirements-dev.txt \
    && rm -rf /root/.cache/pip

RUN pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org sasl \
    && pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org thrift \
    && pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org thrift_sasl

RUN apt-get install -y sasl2-bin

COPY --chown=superset:superset superset superset

ENV PATH=/home/superset/superset/bin:$PATH \
    PYTHONPATH=/home/superset/superset/:$PYTHONPATH

COPY cabenefitfocuscom.crt .
RUN cat cabenefitfocuscom.crt >> /etc/ssl/certs/ca-certificates.crt

USER superset

RUN cd superset/assets \
    && npm ci \
    && npm run sync-backend \
    && npm run build \
    && rm -rf node_modules

COPY contrib/docker/docker-init.sh .
COPY contrib/docker/docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

#HEALTHCHECK CMD ["curl", "-f", "http://localhost:8088/health"]

EXPOSE 8088
