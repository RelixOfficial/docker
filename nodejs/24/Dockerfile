FROM --platform=$TARGETOS/$TARGETARCH node:24-bookworm-slim

LABEL author="RelixOfficial" maintainer="dzakyadis9@gmail.com" description="A Docker image for running Node.js applications."

# Tambah user dan set stop signal
# Buat user dengan shell bash
RUN useradd -m -d /home/container -s /bin/bash container
STOPSIGNAL SIGINT

# ARG Versi Bahasa
ARG GO_VERSION=1.23.10
ARG PYTHON_VERSION=3.12.11
ARG PHP_VERSION=8.2.20
ARG PERL_VERSION=5.38.4

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash \
    curl \
    wget \
    git \
    unzip \
    xz-utils \
    tar \
    zip \
    sudo \
    tzdata \
    build-essential \
    autoconf \
    libtool \
    cmake \
    ffmpeg \
    iproute2 \
    iputils-ping \
    dnsutils \
    sqlite3 \
    libsqlite3-dev \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    liblzma-dev \
    libreadline-dev \
    libffi-dev \
    libgdbm-dev \
    libnss3 \
    libxml2-dev \
    libyaml-dev \
    libjpeg-dev \
    libpng-dev \
    libicu-dev \
    libncurses-dev \
    lsb-release \
    software-properties-common \
    ca-certificates \
    imagemagick \
    webp \
    neofetch \
    pkg-config \
    tini \
    libx11-dev \
    libexpat1-dev \
 && ln -s /bin/bash /usr/bin/bash \
 && rm -rf /var/lib/apt/lists/*

# ====== CACHING FILES ======
ADD https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz /tmp/Python-${PYTHON_VERSION}.tgz
ADD https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz /tmp/go${GO_VERSION}.linux-amd64.tar.gz
ADD https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz /tmp/php-${PHP_VERSION}.tgz
ADD https://www.cpan.org/src/5.0/perl-${PERL_VERSION}.tar.gz /tmp/perl-${PERL_VERSION}.tgz

# ====== GO ======
RUN tar -C /usr/local -xzf /tmp/go${GO_VERSION}.linux-amd64.tar.gz && \
    rm /tmp/go${GO_VERSION}.linux-amd64.tar.gz
ENV PATH="/usr/local/go/bin:$PATH"

# ====== RUST ======
RUN curl https://sh.rustup.rs -sSf | bash -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"

# ====== PERL ======
WORKDIR /tmp/perl
RUN tar -xzf /tmp/perl-${PERL_VERSION}.tgz && \
    cd perl-${PERL_VERSION} && \
    ./Configure -des -Dprefix=/usr/local && \
    make -j$(nproc) && make install && \
    cd / && rm -rf /tmp/perl /tmp/perl-${PERL_VERSION}.tgz

# ====== PHP ======
WORKDIR /tmp/php
RUN tar -xzf /tmp/php-${PHP_VERSION}.tgz && \
    cd php-${PHP_VERSION} && \
    ./configure --disable-cgi --enable-cli --with-openssl --with-zlib && \
    make -j$(nproc) && make install && \
    cd / && rm -rf /tmp/php /tmp/php-${PHP_VERSION}.tgz

# ====== PYTHON ======
WORKDIR /tmp/python
RUN tar -xzf /tmp/Python-${PYTHON_VERSION}.tgz && \
    cd Python-${PYTHON_VERSION} && \
    ./configure --enable-optimizations && \
    make -j$(nproc) && make altinstall && \
    ln -s /usr/local/bin/python3.12 /usr/local/bin/python3 && \
    ln -s /usr/local/bin/python3 /usr/local/bin/python && \
    ln -s /usr/local/bin/pip3.12 /usr/local/bin/pip3 && \
    ln -s /usr/local/bin/pip3 /usr/local/bin/pip && \
    cd / && rm -rf /tmp/python /tmp/Python-${PYTHON_VERSION}.tgz

# ====== Node tools ======
RUN npm install --global npm@latest typescript ts-node @types/node

# ====== pnpm ======
RUN npm install -g corepack && corepack enable && corepack prepare pnpm@latest --activate

# Akses container user
RUN chown -R container:container /home/container

USER container
ENV USER=container HOME=/home/container
WORKDIR /home/container

COPY --chown=container:container ./../entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/entrypoint.sh"]