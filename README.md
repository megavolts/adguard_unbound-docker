Adguard Home Bundled with Unbound DNS
===========

An Alpine Linux Based network-wides ads & trackers blocking DNS serve (AdGuard Home), bundled with a DNSSEC Validating Recursive Resolver (Unbound DNS) docker image.

This image allow to use 127.0.0.1 as un upstream DNS server in Adguard Home configuration

![AdguardHome Version](https://img.shields.io/docker/v/megavolts/adguard_unbound?style=for-the-badge&label=megavolts/adguard_unbound)

![Unbound Version](https://img.shields.io/docker/v/megavolts/unbound?style=for-the-badge&label=megavolts/unbound)

![OpenSSL Version](https://img.shields.io/docker/v/megavolts/openssl-dockerbuildenv?style=for-the-badge&label=megavolts/openssl-dockerbuildenv)

![CD Build Docker Image](https://img.shields.io/github/actions/workflow/status/megavolts/adguard_unbound-docker/CD-30-tag_and_release.yaml?branch=main&style=for-the-badge&label=CD%20Build%20Docker%20Image)

<details> 
    
  <summary>Build status</summary><br>
  <!-- https://img.shields.io/badges/git-hub-actions-workflow-status -->

![CD Unbound Release Check](https://img.shields.io/github/actions/workflow/status/megavolts/adguard_unbound-docker/CD-10-upstream_release_check.yaml?branch=main&style=for-the-badge&label=CD%20Upstream%20Release%20Check)

![CD Build Docker Image](https://img.shields.io/github/actions/workflow/status/megavolts/adguard_unbound-docker/CD-20-build_adguard_unbound.yaml?branch=main&style=for-the-badge&label=CD%20Build%20Docker%20Image)

![CD Release Tag](https://img.shields.io/github/actions/workflow/status/megavolts/adguard_unbound-docker/CD-30-tag_and_release.yaml?branch=main&style=for-the-badge&label=CD%20Release%20Latest%20Tag)

<!-- ![CD Security Scan](https://img.shields.io/github/actions/workflow/status/megavolts/adguard_unbound-docker/CD-90-security_scan.yaml?branch=main&style=for-the-badge&label=CD%20Security%20Scan) -->

</details>

* Adguard Home version: 0.107.67

* Unbound version: 1.24.0

* OpenSSL version: 3.6.0

## Changes
You can see the changes in the [`Releases`](https://github.com/megavolts/adguard_unbound-docker/RELEASES.md) section.


## Architecture
- aarch64
- amd64
- armv7
- armv6


## Available Docker Tags
![Docker Size](https://img.shields.io/docker/image-size/megavolts/adguard_unbound/latest?style=for-the-badge&label=Image%20%20Size)
![Docker Pulls](https://img.shields.io/docker/pulls/megavolts/adguard_unbound?style=for-the-badge&label=Image%20%20Pull)

 You can pull the most recent image from Docker Hub using it's `latest` tag or by using the corresponding image version number:

`docker pull megavolts/adguard_unbound:latest` or `docker pull megavolts/adguard_unbound:0.107.67-0`

We used a complemented dash for any revision, for example `0.107.67-1` to reflect any new relases of OpenSSL or Unbound. 

## DNS over Quic
We are trying to implement [`DNS over Quic`](https://unbound.docs.nlnetlabs.nl/en/latest/topics/privacy/dns-over-quic.html), there will be a canary build for it.

`docker pull megavolts/adguard_unbound:quic`

# Configuration
By default, this image forwards queries Cloudflare DNS server over TLS. In other words, it does not act as a recursive server. The [unbound.sh file]() provides the configuration unless it is overriden as described below. 

This docker image use local root.hints and root.zone file, updated weekly wth a cron jo.b

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

# Acknowledgements

Thanks to the many Docker Images who got me inspired. This code was heavily influenced by [Hat3ph](https://github.com/hat3ph/adguard-unbound), [MatthewVance](https://github.com/MatthewVance/unbound-docker) and team [MΛDИVTTΛH](https://github.com/hat3ph/adguard-unbound).

# Credit:
- [Alpine Linux](https://www.alpinelinux.org/)
- [Docker](https://www.docker.com/)
- [Unbound](https://unbound.net/)
- [OpenSSL](https://www.openssl.org/)
- [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)
- [Aqua Security](https://trivy.dev/)

# Licenses

### License

Unless otherwise specified, all code is released under the MIT License (MIT).
See the [repository's `LICENSE`
file](https://github.com/megavolts/unbound-docker/blob/master/LICENSE) for
details.

### Licenses for other components

- Docker: [Apache 2.0](https://github.com/docker/docker/blob/master/LICENSE)
- OpenSSL: [Apache-style license](https://www.openssl.org/source/license.html)
- Unbound: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)
- AdGuardHome: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)


## TODO
[] add specific configuration under rootfs for adguard_unbound (unbound.sh)