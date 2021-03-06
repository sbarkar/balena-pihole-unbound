FROM balenalib/raspberrypi3-debian:jessie-build AS build

WORKDIR /usr/src

RUN install_packages cmake libraspberrypi-dev

RUN curl -sSL https://raw.githubusercontent.com/tasanakorn/rpi-fbcp/master/CMakeLists.txt -O
RUN curl -sSL https://raw.githubusercontent.com/tasanakorn/rpi-fbcp/master/main.c -O

WORKDIR /usr/src/build

RUN cmake .. && make

FROM pihole/pihole:v5.2.1-armhf-stretch

WORKDIR /usr/src

COPY --from=build /opt/vc/lib/* /opt/vc/lib/
COPY --from=build /usr/src/build/fbcp /usr/src/
COPY services/ /etc/services.d/
COPY init/ /etc/cont-init.d/

ENV DEBIAN_FRONTEND noninteractive

# hadolint ignore=DL3008
RUN apt-get update && \
	apt-get install --no-install-recommends -y console-setup dbus && \
	apt-get clean && rm -rf /var/lib/apt/lists/*

# hadolint ignore=SC2016
RUN sed '/$AUTHORIZED_HOSTNAMES = array(/ a "balena-devices.com",' -i /var/www/html/admin/scripts/pi-hole/php/auth.php

# https://serverfault.com/a/817791
# force dnsmasq to bind only the interfaces it is listening on
# otherwise dnsmasq will fail to start since balena is using 53 on some interfaces
RUN echo "bind-interfaces" >> /etc/dnsmasq.conf

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -fsSL https://github.com/pi-hole/PADD/archive/v3.4.tar.gz | tar xz --strip-components 1 \
	&& chmod +x padd.sh

ENV FONTFACE Terminus
ENV FONTSIZE 8x14