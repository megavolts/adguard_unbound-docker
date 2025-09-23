TODO: Write the readme

Container combining AdGuard Home and Unbound. I don't like the fact you cannot use 127.0.0.1 as an Upstream DNS server when trying to combine these two programs as seperate containers. The only way I found was using the Docker container IP address, which to me isn't reliable enough.

Package version
===========





* Built own [Docker Image - x86_64](https://hub.docker.com/r/megavolts/unbound/tags)

* Updated base image to Alpine: 3.22.1

* AdGuardHome: 0.107.66

* Unbound version: 1.24.0

* OpenSSL version: 3.5.3

## Modification
- Use local root.hints and root.zone file, updated weekly with a cron job

## Available Docker Tags
![Docker Pulls](https://img.shields.io/docker/pulls/megavolts/adguard_unbound)
 You can pull the most recent image from Docker Hub using it's `latest` tag or by using the corresponding image version number:

`docker pull megavolts/adguard_home:latest` or `docker pull megavolts/adguard_home:20200925-0`

The image versioning scheme follows the date-of-release convention, to reflect any new release of OpenSSL, Unbound or AdGuardHome. We used a complemented dash for any image revision, for example `2020095-0`.

![Docker Pulls](https://img.shields.io/docker/pulls/megavolts/adguard_unbound-app)
 There is distroless lightweight builds of the image available. You can pull the image using the `adguard_home-app`, tag are following the full image build.

`docker pull megavolts/adguard_home-app:latest` 

## Available Docker Tags


## DNS over Quic
We are trying to implement [`DNS over Quic`](https://unbound.docs.nlnetlabs.nl/en/latest/topics/privacy/dns-over-quic.html), there will be a canary build for it.

`docker pull madnuttah/unbound:quic`

## Changes and available tags

- [`2025.09.20`, `latest` (*2025.09.20/Dockerfile*)](https://github.com/megavolts/docker-adguard_unbound/tree/master/2025.09.20)



### NEED TO REWRITE ###
You can view the changes in the [`Releases`](https://github.com/megavolts/adguard_home-docker/releases) section.



# Configuration


By default, this image forwards queries Cloudflare DNS server over TLS. In other words, it does not act as a recursive server. The [unbound.sh file]() provides the configuration unless it is overriden as described below. 

The example cutsom_unbound.conf file is different from the one set by unbound.sh file. The example is provided to help you re-configure this as a recursive server. To use it, map '.../custom_unbound.sh' to ..:


# Docker Compose
A `docker-compose.yml` example file is provided.

Use the same volumemappings as the original AdGuardHome container. In fact, you can just swap in this image and everything still works. You only have to update your Upstream DNS server to __127.0.0.1:5053__, which enables Unbound.

As with the original AdGuardHome image, this exposes the following: \



**Volumes:** \
Docker configuration:

## Volumes
* `/opt/adguardhome/work` : working direcotry for AdGuardHome
* `/opt/adguardhome/conf` : configuration folder for AdGuardHome
* `/opt/unbound/conf`: configuration folder for unbound with `unbound.conf`

## Ports
* `53/tcp`: Plain DNS over tcp
* `53/udp`: Plain DNS over ucp
* `67/udp`: DNS only DHCP service over udp
* `68/tcp`: DNS only DHCP service over tcp
* `68/udp`: DNS only DHCP service over udp
* `80/tcp`: Aguard Home admin panel over http
* `443/tcp`: Adguard Home admin panel over https and DNS-over-HTTPS over tcp
* `443/udp`: DNS-over-HTTPS over ucp
* `784/udp`: DNS-over-QUIC
* `853/udp`: DNS-over-TLS
* `3000/tcp`: Aguard Home initial configuration page over http
* `5335/tcp`: Unbound DNS connection over tcp
* `5335/udp`: Unbound DNS connection over ucp

## 


# Acknowledgements

Thanks to the many Docker Images who got me inspired, especiallly to [Hat3ph](https://github.com/hat3ph/adguard-unbound) and team [MΛDИVTTΛH](https://github.com/hat3ph/adguard-unbound).

# Credit:
- [Alpine Linux](https://www.alpinelinux.org/)
- [Docker](https://www.docker.com/)
- [Unbound](https://unbound.net/)
- [OpenSSL](https://www.openssl.org/)
- [Redis](https://redis.io/)
- [Pi-hole](https://pi-hole.net/)
- [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)
- [Aqua Security](https://trivy.dev/)
