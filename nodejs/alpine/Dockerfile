# Gunakan base image Node.js versi 22 yang berbasis Alpine
FROM        --platform=$TARGETOS/$TARGETARCH node:lts-alpine

LABEL       author="RelixOfficial (Diperbaiki oleh Gemini AI)" maintainer="dzakyadis9@gmail.com" description="A Docker image for running Node.js applications with PM2 and essential utilities on Alpine Linux."

# Tambahkan user container dan atur sinyal berhenti
# Gunakan 'adduser' di Alpine, '-D' untuk membuat user sistem tanpa password
RUN adduser -D -h /home/container container
STOPSIGNAL SIGINT

# Atur zona waktu, penting untuk banyak aplikasi
ENV TZ=Asia/Jakarta

# Versi stabil (LTS) untuk tiap bahasa
ARG GO_VERSION=1.23.10
ARG PYTHON_VERSION=3.12.11
ARG PHP_VERSION=8.2.20
ARG PERL_VERSION=5.38.4

RUN apk update && apk upgrade && \
    apk add --no-cache \
      bash curl wget git sqlite sqlite-dev \
      build-base autoconf libtool cmake linux-headers \
      openssl-dev zlib-dev bzip2-dev xz-dev \
      readline-dev ncurses-dev libffi-dev libxml2-dev \
      jpeg-dev libpng-dev icu-dev gcompat \
      tini tzdata shadow sudo util-linux \
      ffmpeg iproute2 elixir libwebp imagemagick \
      openssh bind-tools zip unzip pkgconfig \
      rust cargo \
      coreutils findutils gnupg alpine-sdk \
      py3-wheel py3-setuptools

# ========== Caching Downloads via ADD ==========
# Gunakan ADD untuk mengunduh sumber kode
# Python
ADD https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz /tmp/Python-${PYTHON_VERSION}.tgz
# Go
ADD https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz /tmp/go${GO_VERSION}.linux-amd64.tar.gz
# PHP
ADD https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz /tmp/php-${PHP_VERSION}.tgz
# Perl
ADD https://www.cpan.org/src/5.0/perl-${PERL_VERSION}.tar.gz /tmp/perl-${PERL_VERSION}.tgz


# ========== Install bahasa dari sumber resmi (manual build) ==========

# PYTHON
WORKDIR /tmp
RUN tar -xzf /tmp/Python-${PYTHON_VERSION}.tgz && \
    cd Python-${PYTHON_VERSION} && \
    ./configure --enable-optimizations --prefix=/usr/local && \
    make -j$(nproc) && make altinstall && \
    ln -sf /usr/local/bin/python3.12 /usr/local/bin/python3 && \
    ln -sf /usr/local/bin/python3 /usr/local/bin/python && \
    ln -sf /usr/local/bin/pip3.12 /usr/local/bin/pip3 && \
    ln -sf /usr/local/bin/pip3 /usr/local/bin/pip && \
    cd / && rm -rf /tmp/Python*
    

# GO
# Binary Go biasanya statis dan kompatibel dengan Alpine
RUN tar -C /usr/local -xzf /tmp/go${GO_VERSION}.linux-amd64.tar.gz && \
    rm /tmp/go${GO_VERSION}.linux-amd64.tar.gz
ENV PATH="/usr/local/go/bin:$PATH"

# RUST
# Rustup juga secara otomatis menangani instalasi untuk Alpine
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"

# PERL
WORKDIR /tmp
RUN tar -xzf /tmp/perl-${PERL_VERSION}.tgz && \
    cd perl-${PERL_VERSION} && \
    ./Configure -des -Dprefix=/usr/local && \
    make -j$(nproc) && make install && \
    cd / && rm -rf /tmp/perl*

# PHP
WORKDIR /tmp
RUN tar -xzf /tmp/php-${PHP_VERSION}.tgz && \
    cd php-${PHP_VERSION} && \
    ./configure --disable-cgi --enable-cli --with-openssl --with-zlib --prefix=/usr/local && \
    make -j$(nproc) && make install && \
    cd / && rm -rf /tmp/php*



# ========== Finalisasi dan Pembersihan ==========

# Hapus semua dependensi build yang tidak lagi diperlukan
RUN rm -rf /tmp/* /var/cache/apk/*

# Node.js, PM2, pnpm
RUN npm install --global npm@latest typescript ts-node @types/node && \
    # pnpm
    npm install -g corepack && corepack enable && corepack prepare pnpm@latest --activate

# Ganti kepemilikan direktori home ke user container
RUN chown -R container:container /home/container

# Ganti ke user non-root
USER container
ENV USER=container HOME=/home/container
WORKDIR /home/container

COPY --chown=container:container ./../entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/entrypoint.sh"]