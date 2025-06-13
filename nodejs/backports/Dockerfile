FROM        --platform=$TARGETOS/$TARGETARCH node:lts-bullseye-slim

LABEL author="RelixOfficial" maintainer="dzakyadis9@gmail.com" description="A Docker image for running Node.js applications with PM2 and essential utilities."

# Create non-root user
RUN useradd -m -d /home/container container
STOPSIGNAL SIGINT

ARG GO_VERSION=1.23.10
ARG PYTHON_VERSION=3.12.11
ARG PHP_VERSION=8.2.20
ARG PERL_VERSION=5.38.4

# Enable backports
RUN echo 'deb http://deb.debian.org/debian bullseye-backports main contrib non-free' > /etc/apt/sources.list.d/backports.list && \
    apt update

# Install base dependencies (use backports selectively)
RUN apt -y install -t bullseye-backports \
    ffmpeg \
    git \
    libssl-dev \
    libnss3 \
    tini \
    imagemagick && \
    apt -y install \
    iproute2 \
    sqlite3 \
    libsqlite3-dev \
    elixir \
    webp \
    neofetch \
    ssh \
    sudo \
    wget \
    ca-certificates \
    dnsutils \
    tzdata \
    zip \
    tar \
    curl \
    build-essential \
    libtool \
    iputils-ping \
    unzip \
    xz-utils \
    pkg-config \
    zlib1g-dev \
    libreadline-dev \
    libbz2-dev \
    liblzma-dev \
    libncurses-dev \
    libgdbm-dev \
    libffi-dev \
    libmpdec-dev \
    libicu-dev \
    libpq-dev \
    cmake \
    autoconf \
    libxml2-dev \
    libyaml-dev \
    libjpeg-dev \
    libpng-dev \
    gnupg \
    lsb-release \
    software-properties-common

# Python
ADD https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz /tmp/Python-${PYTHON_VERSION}.tgz
# Go
ADD https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz /tmp/go${GO_VERSION}.linux-amd64.tar.gz
# PHP
ADD https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz /tmp/php-${PHP_VERSION}.tgz
# Perl
ADD https://www.cpan.org/src/5.0/perl-${PERL_VERSION}.tar.gz /tmp/perl-${PERL_VERSION}.tgz

# GO
RUN tar -C /usr/local -xzf /tmp/go${GO_VERSION}.linux-amd64.tar.gz && \
    rm /tmp/go${GO_VERSION}.linux-amd64.tar.gz
ENV PATH="/usr/local/go/bin:$PATH"

# RUST
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"

# PERL
WORKDIR /tmp/perl
RUN tar -xzf /tmp/perl-${PERL_VERSION}.tgz && \
    cd perl-${PERL_VERSION} && \
    ./Configure -des -Dprefix=/usr/local && \
    make -j$(nproc) && make install && \
    rm -rf /tmp/perl

# PHP
WORKDIR /tmp/php
RUN tar -xzf /tmp/php-${PHP_VERSION}.tgz && \
    cd php-${PHP_VERSION} && \
    ./configure --disable-cgi --enable-cli --with-openssl --with-zlib && \
    make -j$(nproc) && make install && \
    rm -rf /tmp/php

# PYTHON
WORKDIR /tmp/python
RUN tar -xzf /tmp/Python-${PYTHON_VERSION}.tgz && \
    cd Python-${PYTHON_VERSION} && \
    ./configure --enable-optimizations && \
    make -j$(nproc) && make altinstall && \
    ln -s /usr/local/bin/python3.12 /usr/local/bin/python3 && \
    ln -s /usr/local/bin/python3 /usr/local/bin/python && \
    ln -s /usr/local/bin/pip3.12 /usr/local/bin/pip3 && \
    ln -s /usr/local/bin/pip3 /usr/local/bin/pip && \
    rm -rf /tmp/python

# Node.js Tooling
RUN npm install --global npm@latest typescript ts-node @types/node


# pnpm
RUN npm install -g corepack && corepack enable && corepack prepare pnpm@latest --activate

# Permissions
RUN chown -R container:container /home/container

USER container
ENV USER=container HOME=/home/container
WORKDIR /home/container

COPY --chown=container:container ./../entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/entrypoint.sh"]