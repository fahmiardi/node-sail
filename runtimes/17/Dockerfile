FROM amd64/ubuntu:21.10

ARG WWWGROUP
ARG NODE_VERSION=17
ARG NPM_AUTH

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /code

RUN apt-get update \
    && apt-get install -y gosu curl wget zip unzip git supervisor libaio1 \
    && wget https://download.oracle.com/otn_software/linux/instantclient/215000/instantclient-basic-linux.x64-21.5.0.0.0dbru.zip -O /tmp/instantclient.zip \
    && unzip /tmp/instantclient.zip -d /opt/ \
    && sh -c 'echo /opt/instantclient_21_5/ > /etc/ld.so.conf.d/oracle.conf' \
    && ldconfig \
    && curl -sL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm ts-node-dev \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail

RUN echo "PATH=/opt/instantclient_21_5:$PATH" >> /home/sail/.bashrc \
    && echo "LD_LIBRARY_PATH=/opt/instantclient_21_5" >> /home/sail/.bashrc

RUN sh -c 'echo ${NPM_AUTH} > /home/sail/.npmrc' \
    && chown sail:dialout /home/sail/.npmrc

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod +x /usr/local/bin/start-container

EXPOSE 8000

ENTRYPOINT ["start-container"]