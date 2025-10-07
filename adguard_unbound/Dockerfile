ARG BUILD_REPOSITORY="megavolts/adguard_unbound" \
    BUILD_NAME="AdGuard Unbound" \
    BUILD_DESCRIPTION="Unbound is a validating, recursive, and caching DNS resolver. Bundled with AdGuard" \
    BUILD_DATE \
    ADGUARDHOME_VERSION \
    UNBOUND_DOCKERBUILDENV_VERSION \
    DOCKER_IMAGE_VERSION

# 1. Stage Unbound and Adguard Home
## 1.1 Checkout unbound
FROM megavolts/unbound:$UNBOUND_DOCKERBUILDENV_VERSION AS unbound

## 1.2. Install AdGuardHome
FROM --platform=$BUILDPLATFORM alpine:latest AS adguard

WORKDIR /tmp/src

ARG ADGUARDHOME_VERSION \
    TARGETARCH

RUN set -xe; apk --update --no-cache add \
        curl \
    && if [[ "${TARGETARCH}" = "aarch64" ]]; then ARCH="arm64"; \
       elif [[ "${TARGETARCH}" = "arm/v7" ]]; then ARCH="armv7"; \
       elif [[ "${TARGETARCH}" = "arm/v6" ]]; then ARCH="armv6"; \
       else ARCH=${TARGETARCH}; fi \
    && curl -L https://github.com/AdguardTeam/AdGuardHome/releases/download/v${ADGUARDHOME_VERSION}/AdGuardHome_linux_${ARCH}.tar.gz -o adguard.tar.gz \
    && mkdir -p \
        /usr/local/adguardhome/sbin \
        /usr/local/adguardhome/conf \
        /usr/local/adguardhome/work \
    && tar -zxf adguard.tar.gz ./AdGuardHome/AdGuardHome  --strip-components=2 -C /usr/local/adguardhome/sbin  \
    && chown -R nobody:nogroup /usr/local/adguardhome/sbin \
    && chmod 0700 /usr/local/adguardhome/sbin \
    && chmod 0700 /usr/local/adguardhome/work

# # 2. Create adguard_unbound container
# # 2.1 Copy files from adgaurd and unbound
FROM scratch AS stage

# copy adguard
COPY --from=adguard /usr/local/adguardhome \
        /app/usr/local/adguardhome

# copy unbound
COPY --from=unbound /usr/local/unbound \
        /app/usr/local/unbound

# copy  to /sbin        
COPY --from=unbound /sbin \
        /app/sbin     
# copy  to /usr/sbin
COPY --from=unbound /usr/sbin \
        /app/usr/sbin  

# copy to /etc
COPY --from=unbound /etc/crontabs/root \
        /app/etc/crontabs/root
COPY --from=unbound /etc/ssl/ \
        /app/etc/ssl/
COPY --from=unbound /etc/passwd /etc/group \
        /app/etc/

# copy to /lib
COPY --from=unbound /lib \
        /app/lib/
# copty to /usr/lib
COPY --from=unbound /usr/lib \
        /app/usr/lib

# copy to /bin
COPY --from=unbound /bin \
        /app/bin/
# copy to /usr/bin
COPY --from=unbound /usr/bin \
        /app/usr/bin/
 
# timezone
COPY --from=unbound /usr/share/zoneinfo/ \
        /app/usr/share/zoneinfo/

# entrypoint
COPY --from=unbound --chmod=755 /entrypoint \
        /app/

## 2.2 Assemble adguard_unbound
WORKDIR /

FROM scratch AS adguard_unbound-app

ARG ADGUARDHOME_VERSION \
    BUILD_REVISION \
    BUILD_DATE \
    BUILD_TITLE \
    BUILD_DESCRIPTION \
    BUILD_NAME \
    BUILD_REPOSITORY \
    DOCKER_IMAGE_VERSION \
    TARGETARCH
    
WORKDIR /

VOLUME ["/usr/local/adguardhome/conf", "/usr/local/adguardhome/work", "/usr/local/unbound"]

ENV UNBOUND_HEALTHCHECK_PORT=${UNBOUND_HEALTHCHECK_PORT} \
    DISABLE_SET_PERMS=${DISABLE_SET_PERMS} \
    UNBOUND_UID=${UNBOUND_UID} \
    UNBOUND_GID=${UNBOUND_GID} \
    TZ=${TZ} \
    PATH=/usr/local/unbound/unbound.d/sbin:/usr/local/adguardhome/sbin:"$PATH" 

COPY --from=stage /app/ /

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
      org.opencontainers.image.url="https://github.com/${BUILD_REPOSITORY}" \
      org.opencontainers.image.vendor="Marc Oggier" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
      org.opencontainers.image.created="${BUILD_DATE}"