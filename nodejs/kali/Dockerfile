FROM        --platform=$TARGETOS/$TARGETARCH kalilinux/kali-rolling

LABEL author="RelixOfficial" maintainer="dzakyadis9@gmail.com" description="A Docker image for running Node.js apps on Kali Linux with PM2 and essential utilities."

# Tambahkan user container dan stop signal
RUN useradd -m -d /home/container container
STOPSIGNAL SIGINT

# Versi stabil (LTS) untuk tiap bahasa
ARG GO_VERSION=1.23.10
ARG PYTHON_VERSION=3.12.11
ARG PHP_VERSION=8.1.28
ARG PERL_VERSION=5.38.4


# Update sistem dan install dependencies + tools dari Kali
RUN apt update && apt -y full-upgrade && \
    apt -y install \
    ffmpeg \
    iproute2 \
    git \
    sqlite3 \
    libsqlite3-dev \
    elixir \
    webp \
    imagemagick \
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
    libnss3 \
    tini \
    unzip \
    xz-utils \
    pkg-config \
    libssl-dev \
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
    software-properties-common \
    nmap \
    hydra \
    metasploit-framework \
    burpsuite && \
    git clone https://github.com/dylanaraps/neofetch.git /opt/neofetch && \
    ln -s /opt/neofetch/neofetch /usr/bin/neofetch

# Tambah Node.js LTS dari NodeSource
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt install -y nodejs && \
    npm install -g npm@latest typescript ts-node @types/node pm2 corepack && \
    corepack enable && corepack prepare pnpm@latest --activate

# ========== Caching Downloads via ADD ==========
ADD https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz /tmp/Python-${PYTHON_VERSION}.tgz
ADD https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz /tmp/go${GO_VERSION}.linux-amd64.tar.gz
ADD https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz /tmp/php-${PHP_VERSION}.tgz
ADD https://www.cpan.org/src/5.0/perl-${PERL_VERSION}.tar.gz /tmp/perl-${PERL_VERSION}.tgz

# ========== Install bahasa dari sumber resmi (manual build) ==========


# GO
RUN tar -C /usr/local -xzf /tmp/go${GO_VERSION}.linux-amd64.tar.gz && rm /tmp/go${GO_VERSION}.linux-amd64.tar.gz
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
    
RUN npm install -g npm@latest typescript ts-node @types/node
RUN npm install -g corepack && corepack enable && corepack prepare pnpm@latest --activate

# Beri akses penuh ke user container
RUN chown -R container:container /home/container
RUN chmod 755 /home/container

USER container
ENV USER=container HOME=/home/container
WORKDIR /home/container

COPY --chown=container:container ./../entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/entrypoint.sh"]