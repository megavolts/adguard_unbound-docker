
ARG BUILD_REPOSITORY="megavolts/adguard_unbound" \
    BUILD_NAME="AdGuard Unbound" \
    BUILD_DESCRIPTION="Unbound is a validating, recursive, and caching DNS resolver. Bundled with AdGuard" \
    BUILD_DATE="2025-09-19" \
    UNBOUND_VERSION="1.24.0" \
    ADGUARD_VERSION="0.107.66" \
    OPENSSL_VERSION="3.5.3" \
    BUILD_REVISION=0 \
    UNBOUND_UID="1000" \
    UNBOUND_GID="1000"


# Stage 1: build adguard_unbound docker
## 1.1. Build OpenSSL
FROM --platform=$BUILDPLATFORM alpine:latest AS buildenv

FROM buildenv AS openssl

# Define argument
ARG OPENSSL_VERSION

# Define environment variables
ENV OPENSSL_VERSION=${OPENSSL_VERSION} \
    OPENSSL_PGP="BA5473A2B0587B07FB27CF2D216094DFD0CB81EF" \
    OPENSSL_DOWNLOAD_URL="https://github.com/openssl/openssl/releases/download/openssl-"${OPENSSL_VERSION}"/openssl-"${OPENSSL_VERSION}".tar.gz"

WORKDIR /tmp/src

# Build OpenSSl from sources with checksum
RUN set -xe; \
    apk --no-cache --update add \
        ca-certificates \
        curl \
        file \
        gnupg \
    && apk --update add --no-cache --virtual .build-deps \
        build-base \
        perl \
        libevent-dev \
        libidn2-dev \
        linux-headers \
        apk-tools \
    && curl -sSL ${OPENSSL_DOWNLOAD_URL} -o openssl-${OPENSSL_VERSION}.tar.gz \
    && curl -sSL ${OPENSSL_DOWNLOAD_URL}.sha256 -o openssl-${OPENSSL_VERSION}.tar.gz.sha256 \
    && curl -sSL ${OPENSSL_DOWNLOAD_URL}.asc -o openssl-${OPENSSL_VERSION}.tar.gz.asc \
    && echo "`cat openssl-${OPENSSL_VERSION}.tar.gz.sha256`" | sha256sum -c - \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && gpg --no-tty --keyserver hkps://keys.openpgp.org --recv-keys ${OPENSSL_PGP} \
    && gpg --batch --verify openssl-${OPENSSL_VERSION}.tar.gz.asc openssl-${OPENSSL_VERSION}.tar.gz \
    && pkill -9 gpg-agent \
    && pkill -9 dirmngr \
    && tar xzf openssl-${OPENSSL_VERSION}.tar.gz \
    && rm openssl-${OPENSSL_VERSION}.tar.gz \
    && cd openssl-${OPENSSL_VERSION} \
    && ./Configure \
        no-weak-ssl-ciphers \
        no-apps \
        no-docs \
        no-legacy \
        no-ssl3 \
        no-err \
        no-autoerrinit \
        enable-tfo \
        enable-quic \
        enable-ktls \
        enable-ec_nistp_64_gcc_128 \
        -fPIC \
        -DOPENSSL_NO_HEARTBEATS \
        -fstack-protector-strong \
        -fstack-clash-protection \
        --prefix=/usr/local/openssl \
        --openssldir=/usr/local/openssl \
        --libdir=/usr/local/openssl/lib \
    && make \
    && make install_sw \
    && apk del --no-cache .build-deps \
    && rm -rf \
        /usr/share/man \
        /usr/share/docs \
        /usr/local/openssl/bin \
        /tmp/* \
        /var/tmp/* \
        /var/log/* 


## 1.2 Build Unbound with latest OpenSSL
FROM openssl AS unbound

# Define argument
ARG UNBOUND_VERSION \
    UNBOUND_UID \
    UNBOUND_GID

# Define environment
ENV UNBOUND_VERSION="${UNBOUND_VERSION}" \
    UNBOUND_DOWNLOAD_URL="https://www.nlnetlabs.nl/downloads/unbound/unbound-"${UNBOUND_VERSION}".tar.gz" \
    INTERNIC_PGP="F0CB1A326BDF3F3EFA3A01FA937BB869E3A238C5" \
    UNBOUND_PGP_WIJNGAARDS="EDFAA3F2CA4E6EB05681AF8E9F6F1C2D7E045F8D" \
    UNBOUND_PGP_GEORGE="948EB42322C5D00B79340F5DCFF3344D9087A490" \
    UNBOUND_UID="${UNBOUND_UID}" \
    UNBOUND_GID="${UNBOUND_GID}"

# Build unbound from sources with checksum
RUN set -xe; \
      addgroup -S -g "${UNBOUND_GID}" _unbound \
    && adduser -S -H -h /usr/local/unbound -g _unbound -u "${UNBOUND_UID}" -D -G _unbound _unbound \
    && apk --update --no-cache add \
        ca-certificates \
        gnupg \
        curl \
        file \
        binutils \
    && apk --update --no-cache add --virtual .build-deps \
        build-base \
        libevent-dev \
        libsodium-dev \
        linux-headers \
        nghttp2-dev \
        ngtcp2-dev \
        expat-dev \
        protobuf-c-dev \
        hiredis-dev \
        bash \
    && curl -sSL ${UNBOUND_DOWNLOAD_URL} -o unbound.tar.gz \
    && curl -sSL ${UNBOUND_DOWNLOAD_URL}.asc -o unbound.tar.gz.asc \
    && curl -sSL ${UNBOUND_DOWNLOAD_URL}.sha256 -o unbound.tar.gz.sha256 \
    && echo "`cat unbound.tar.gz.sha256` unbound.tar.gz" | sha256sum -c - \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
#    && gpg --no-tty --keyserver hkps://keys.openpgp.org --recv-keys "${UNBOUND_PGP_WIJNGAARDS}" "${UNBOUND_PGP_GEORGE}" \
    && gpg --no-tty --recv-keys "${UNBOUND_PGP_WIJNGAARDS}" "${UNBOUND_PGP_GEORGE}" \
    && gpg --batch --verify unbound.tar.gz.asc unbound.tar.gz \
    && tar -xzf unbound.tar.gz \
    && rm unbound.tar.gz \
    && cd unbound-"${UNBOUND_VERSION}" \
    && ./configure \
        --prefix=/usr/local/unbound/unbound.d/ \
        --with-run-dir=/usr/local/unbound/unbound.d \
        --with-conf-file=/usr/local/unbound/unbound.conf \
        --with-pidfile=/usr/local/unbound/unbound.d/unbound.pid \
        --mandir=/usr/share/man \
        --with-rootkey-file=/usr/local/unbound/iana.d/root.key \
        --with-ssl=/usr/local/openssl \
        --with-libevent \
        --with-libnghttp2 \
        --with-libhiredis \
        --with-username=_unbound \
        --disable-shared \
        --enable-dnstap \
        --enable-dnscrypt \
        --enable-cachedb \
        --enable-subnet \
        --with-pthreads \
        --without-pythonmodule \
        --without-pyunbound \
        --enable-event-api \
        --enable-tfo-server \
        --enable-tfo-client \
        --enable-pie \
        --enable-relro-now \
    && make \
    && make install \
    && mkdir -p "/usr/local/unbound/iana.d/" \
    && curl -sSL https://www.internic.net/domain/named.cache -o /usr/local/unbound/iana.d/root.hints \
    && curl -sSL https://www.internic.net/domain/named.cache.md5 -o /usr/local/unbound/iana.d/root.hints.md5 \
    && curl -sSL https://www.internic.net/domain/named.cache.sig -o /usr/local/unbound/iana.d/root.hints.sig \
    && echo "`cat /usr/local/unbound/iana.d/root.hints.md5` /usr/local/unbound/iana.d/root.hints" | md5sum -c - \
    && curl -sSL https://www.internic.net/domain/root.zone -o /usr/local/unbound/iana.d/root.zone \
    && curl -sSL https://www.internic.net/domain/root.zone.md5 -o /usr/local/unbound/iana.d/root.zone.md5 \
    && curl -sSL https://www.internic.net/domain/root.zone.sig -o /usr/local/unbound/iana.d/root.zone.sig \
    && echo "`cat /usr/local/unbound/iana.d/root.zone.md5` /usr/local/unbound/iana.d/root.zone" | md5sum -c - \
    && GNUPGHOME="$(mktemp -d)"\
    && export GNUPGHOME \
    && gpg --no-tty --recv-keys "$INTERNIC_PGP" \
    && gpg --verify /usr/local/unbound/iana.d/root.hints.sig /usr/local/unbound/iana.d/root.hints \
    && gpg --verify /usr/local/unbound/iana.d/root.zone.sig /usr/local/unbound/iana.d/root.zone \
    && /usr/local/unbound/sbin/unbound-anchor -v -a /usr/local/unbound/iana.d/root.key || true \
    && pkill -9 gpg-agent\
    && pkill -9 dirmngr

COPY ./rootfs/ /

RUN mkdir -p \   
        /usr/local/unbound/conf.d/ \
        /usr/local/unbound/certs.d/ \
        /usr/local/unbound/zones.d/ \
        /usr/local/unbound/log.d/ \
    && touch /usr/local/unbound/log.d/unbound.log \
    && chown -R _unbound:_unbound \
        /usr/local/unbound/ \
    && ln -s /dev/random /dev/urandom /dev/null \
        /usr/local/unbound/unbound.d/ \
    && chown -Rh _unbound:_unbound \
        /usr/local/unbound/unbound.d/random \
        /usr/local/unbound/unbound.d/null \
        /usr/local/unbound/unbound.d/urandom \
    && chmod -R 770 \
        /usr/local/unbound/sbin/* \
    && rm -rf \
        /usr/local/unbound/unbound.conf \
        /usr/local/unbound/unbound.d/share \
        /usr/local/unbound/etc \
        /usr/local/unbound/iana.d/root.hints.* \
        /usr/local/unbound/iana.d/root.zone.* \
        /usr/local/unbound/unbound.d/include \
        /usr/local/unbound/unbound.d/lib \
    && find /usr/local/openssl/lib/libssl.so.* -type f | xargs strip --strip-all \
    && find /usr/local/openssl/lib/libcrypto.so.* -type f | xargs strip --strip-all \
    && strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound \
    && strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-anchor \
    && strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-checkconf \
    && strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-control \
    && strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-host
    # enable root.hints and root.zone updated via crontab
    && (crontab -l 2>/dev/null; echo "0 0 * * 0 /bin/bash /usr/local/unbound/sbin/01-update_root_hints.sh && unbound-control reload") | crontab - \
    && (crontab -l 2>/dev/null; echo "0 0 * * 0 /bin/bash /usr/local/unbound/sbin/01-update_root_zone.sh && unbound-control reload") | crontab - \
    && (crontab -l 2>/dev/null; echo "5 0 * * 0 /bin/bash /usr/local/unbound/sbin/02-check_signature.sh && unbound-control reload") | crontab - \

COPY ./rootfs/usr/local/unbound/unbound.conf /usr/local/unbound/unbound.conf


## 1.3. Install AdGuardHome
FROM buildenv AS adguard

WORKDIR /tmp/src

ARG ADGUARD_VERSION \
    TARGETARCH

RUN set -xe; apk --update --no-cache add \
        curl \
    && if [[ "${TARGETARCH}" = "aarch64" ]]; then ARCH="arm64"; fi \
    && if [[ "${TARGETARCH}" = "arm64" ]]; then ARCH="arm64"; fi \
    && if [[ "${TARGETARCH}" = "amd64" ]]; then ARCH="amd64"; fi \
    && if [[ "${TARGETARCH}" = "armv7" ]]; then ARCH="armv7"; fi \
    && curl -L https://github.com/AdguardTeam/AdGuardHome/releases/download/v${ADGUARD_VERSION}/AdGuardHome_linux_${ARCH}.tar.gz -o adguard.tar.gz \
    && mkdir -p \
        /usr/local/adguardhome/sbin \
        /usr/local/adguardhome/conf \
        /usr/local/adguardhome/work \
    && tar -zxvf adguard.tar.gz ./AdGuardHome/AdGuardHome  --strip-components=2 -C /usr/local/adguardhome/sbin  \
    && chown -R nobody:nogroup /usr/local/adguardhome/sbin \
    && chmod 0700 /usr/local/adguardhome/sbin \
    && chmod 0700 /usr/local/adguardhome/work


## 1.4 Assemble for target platform
FROM alpine AS stage

COPY --from=adguard /usr/local/adguardhome /usr/local/adguardhome
COPY --from=unbound /usr/local/unbound /usr/local/unbound
COPY --from=unbound /usr/local/openssl /usr/local/openssl
COPY --from=unbound /usr/local/sbin /usr/local/sbin
COPY --from=unbound /etc/passwd /etc/group /etc/
COPY --from=unbound /etc/crontabs/root /etc/crontabs/root
COPY --from=unbound --chmod=775 /usr/local/sbin/* /usr/local/sbin/
COPY --from=unbound --chmod=775 /entrypoint /

RUN set -xe; \
    apk --update --no-cache add \
    ca-certificates \
    # to update root.*
    curl \
    gpg \
    perl \
    # to tune unbound conf
    coreutils \
    libattr \
    utmps-libs \
    skalibs-libs \
    # to execute unbound
    expat \
    protobuf-c \
    libsodium \
    nghttp2 \
    libevent \
    hiredis \
    ngtcp2 \
    # for timezone
    tzdata \
    # for dig
    drill \
    tini \
    # to execute unbound.sh
    shadow \
    su-exec \
    bash \
    #cleanup
    && rm -rf /tmp/* /var/cache/apk/*

    
## 1.5 Finalize
FROM stage AS adguard_unbound

ARG BUILD_REVISION \
    BUILD_DATE \
    BUILD_TITLE \
    BUILD_DESCRIPTION \
    BUILD_NAME \
    BUILD_REPOSITORY \
    TARGETARCH \
    DOCKER_IMAGE_VERSION=${BUILD_DATE}-${BUILD_REVISION}

ENV UNBOUND_HEALTHCHECK_PORT=${UNBOUND_HEALTHCHECK_PORT} \
    DISABLE_SET_PERMS=${DISABLE_SET_PERMS} \
    UNBOUND_UID=${UNBOUND_UID} \
    UNBOUND_GID=${UNBOUND_GID} \
    TZ=${TZ} \
    PATH=/usr/local/unbound/unbound.d/sbin:/usr/local/adguardhome/sbin:"$PATH" 

WORKDIR /

VOLUME ["/usr/local/adguardhome/conf", "/usr/local/adguardhome/work", "/usr/local/unbound"]

# 53     : TCP, UDP : DNS
# 67     :      UDP : DHCP (server)
# 68     :      UDP : DHCP (client)
# 80     : TCP      : HTTP (main)
# 443    : TCP, UDP : HTTPS, DNS-over-HTTPS (incl. HTTP/3), DNSCrypt (main)
# 853    : TCP, UDP : DNS-over-TLS, DNS-over-QUIC
# 3000   : TCP, UDP : HTTP(S) (alt, incl. HTTP/3) for initial configuration
# 5335   : TCP, UDP : Unbound DNS resolver
# 5443   : TCP, UDP : DNSCrypt (alt)
# 6060   : TCP      : HTTP (pprof)
EXPOSE 53/tcp 53/udp
EXPOSE 67/udp
EXPOSE 68/udp 68/tcp
EXPOSE 80/tcp
EXPOSE 443/tcp 443/udp
EXPOSE 784/udp
EXPOSE 853/udp
EXPOSE 3000/tcp
EXPOSE 5335/udp 5335/tcp
EXPOSE 5443/tcp 5443/udp
EXPOSE 6060/tcp

HEALTHCHECK --interval=30s --timeout=15s --start-period=5s CMD sh /usr/local/sbin/healthcheck.sh 

ENTRYPOINT ["/sbin/tini", "--", "/entrypoint"]

LABEL maintainer="megavolts <marc.oggier@megavolts.ch>"\
      org.opencontainers.image.arch="${TARGETARCH}" \
      org.opencontainers.image.version=${DOCKER_IMAGE_VERSION} \
      org.opencontainers.image.title="BUILD_NAME" \
      org.opencontainers.image.description="BUILD_DESCRIPTION" \
      org.opencontainers.image.url="https://github.com/"${BUILD_REPOSITORY} \
      org.opencontainers.image.vendor="Marc Oggier" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.source="https://github.com/"${BUILD_REPOSITORY}

# # 2. Stage 2 - Make distroless adguard_unbound-app
# # 2.1 Copy files from adguard_unbound
# # aka make it tiny
FROM scratch AS stage_app

COPY --from=adguard_unbound /usr/local/unbound \
        /app/usr/local/unbound

COPY --from=adguard_unbound /usr/local/sbin \
        /app/usr/local/sbin        

COPY --from=adguard_unbound /etc/crontabs/root \
        /app/etc/crontabs/root

COPY --from=adguard_unbound /usr/local/adguardhome \
        /app/usr/local/adguardhome

COPY --from=adguard_unbound /lib/*-musl-* \
        /app/lib/

COPY --from=adguard_unbound /bin/sh /bin/sed /bin/grep /bin/netstat /bin/chown /bin/chgrp \
        /app/bin/
  
COPY --from=adguard_unbound /sbin/su-exec /sbin/tini \
        /app/sbin/
  
# for changing user
COPY --from=adguard_unbound /usr/sbin/groupmod /usr/sbin/usermod \
        /app/usr/sbin/

# for root.* updated
COPY --from=adguard_unbound /usr/bin/curl /usr/bin/gpg /usr/bin/dirmngr /usr/bin/md5sum /usr/bin/nproc /usr/bin/perl \
        /app/usr/bin/
    
# for tuning unbound
COPY --from=adguard_unbound  /usr/bin/nproc /usr/bin/perl \
        /app/usr/bin/

COPY --from=adguard_unbound /usr/lib/libacl* /usr/lib/libattr* /usr/lib/libutmps*  /usr/lib/libskarnet* \
        /app/usr/lib/

COPY --from=adguard_unbound  /usr/lib/perl5/core_perl/CORE/libperl* \
        /app/usr/lib/perl5/core_perl/CORE/

# for healthcheck.sh
COPY --from=adguard_unbound /usr/bin/awk /usr/bin/drill /usr/bin/id \
        /app/usr/bin/

# for ssl
COPY --from=adguard_unbound /usr/local/openssl/lib/libssl.so.* /usr/local/openssl/lib/libcrypto.so.* \
        /app/lib/
  
COPY --from=adguard_unbound /usr/lib/libgcc_s* \
        /usr/lib/libbsd* \ 
        /usr/lib/libmd* \
        /usr/lib/libsodium* \
        /usr/lib/libexpat* \
        /usr/lib/libprotobuf-c* \
        /usr/lib/libnghttp2* \
        /usr/lib/libldns* \
        /usr/lib/libhiredis* \
        /usr/lib/libevent* \
        /usr/lib/libngtcp2* \
        /app/usr/lib/
 
COPY --from=adguard_unbound /etc/ssl/ \
        /app/etc/ssl/

# user and groups
COPY --from=adguard_unbound /etc/passwd /etc/group \
        /app/etc/
  
# timezone
COPY --from=adguard_unbound /usr/share/zoneinfo/ \
        /app/usr/share/zoneinfo/

# entrypoint
COPY --from=adguard_unbound --chmod=755 /entrypoint \
        /app/

## 2.2 Assemble adguard_unbound-app
WORKDIR /

FROM scratch AS adguard_unbound-app

ARG UNBOUND_VERSION \
    ADGUARD_VERSION \
    BUILD_REVISION \
    BUILD_DATE \
    BUILD_TITLE \
    BUILD_DESCRIPTION \
    BUILD_NAME \
    BUILD_REPOSITORY \
    DOCKER_IMAGE_VERSION=${BUILD_DATE}-${BUILD_REVISION} \
    TARGETARCH
    
WORKDIR /

VOLUME ["/usr/local/adguardhome/conf", "/usr/local/adguardhome/work", "/usr/local/unbound"]

ENV UNBOUND_HEALTHCHECK_PORT=${UNBOUND_HEALTHCHECK_PORT} \
    DISABLE_SET_PERMS=${DISABLE_SET_PERMS} \
    UNBOUND_UID=${UNBOUND_UID} \
    UNBOUND_GID=${UNBOUND_GID} \
    TZ=${TZ} \
    PATH=/usr/local/unbound/unbound.d/sbin:/usr/local/adguardhome/sbin:"$PATH" 

COPY --from=stage_app /app/ /

HEALTHCHECK --interval=30s --timeout=15s --start-period=5s CMD sh /usr/local/sbin/healthcheck.sh 

ENTRYPOINT ["/sbin/tini", "--", "/entrypoint"]

LABEL maintainer="megavolts <marc.oggier@megavolts.ch>"\
      org.opencontainers.image.arch="${TARGETARCH}" \
      org.opencontainers.image.version=${DOCKER_IMAGE_VERSION} \
      org.opencontainers.image.title="BUILD_NAME"-app \
      org.opencontainers.image.description="BUILD_DESCRIPTION" \
      org.opencontainers.image.url="https://github.com/"${BUILD_REPOSITORY} \
      org.opencontainers.image.vendor="Marc Oggier" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.source="https://github.com/"${BUILD_REPOSITORY}
