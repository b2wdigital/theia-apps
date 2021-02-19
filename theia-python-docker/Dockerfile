ARG NODE_VERSION=12.18.3

FROM node:${NODE_VERSION}
ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
RUN useradd -ms /bin/bash theia && \
    chown -R theia /home/
USER theia
ENV PS1 "$> "
RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean

FROM node:${NODE_VERSION}

# Install Python 3 from source
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y make build-essential libssl-dev \
    && apt-get install -y libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    && apt-get install -y libncurses5-dev  libncursesw5-dev xz-utils tk-dev \
    && apt-get install -y python3 python3-dev python3-pip \
    && pip3 install --upgrade --user pip python-language-server flake8 autopep8 pylint jupyter virtualenv \
    && apt-get install -y yarn \
    && apt-get clean \
    && apt-get auto-remove -y \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* \
    && alias python=python3 \
    && alias pip=pip3

RUN mkdir -p /home/theia \
    && mkdir -p /home/project
ENV HOME /home/theia
WORKDIR /home/theia
COPY --from=0 /home/theia /home/theia
EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins
ENV USE_LOCAL_GIT true
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
