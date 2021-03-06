FROM ubuntu-debootstrap:14.04
MAINTAINER dreamcat4 <dreamcat4@gmail.com>


ENV _clean="rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*"
ENV _apt_clean="eval apt-get clean && $_clean"


# Install s6-overlay
ENV s6_overlay_version="1.13.0.0"
ADD https://github.com/just-containers/s6-overlay/releases/download/v${s6_overlay_version}/s6-overlay-amd64.tar.gz /tmp/
RUN tar zxf /tmp/s6-overlay-amd64.tar.gz -C / && $_clean
ENV S6_LOGGING="1"
# ENV S6_KILL_GRACETIME="3000"


# Install pipework
ADD https://github.com/jpetazzo/pipework/archive/master.tar.gz /tmp/pipework-master.tar.gz
RUN tar -zxf /tmp/pipework-master.tar.gz -C /tmp && cp /tmp/pipework-master/pipework /sbin/ && $_clean


# Install deluge
RUN apt-get update -qq && DEBIAN_FRONTEND=noninteractive \
    apt-get install -qqy sudo socat deluged deluge-web deluge-console \
 && $_apt_clean


# Relocate the timezone file
RUN mkdir -p /config/.etc && mv /etc/timezone /config/.etc/ && ln -s /config/.etc/timezone /etc/


# Relocate the locale files
RUN mkdir -p /config/.var/lib/locales/ /config/.usr/lib/ \
 && mv /var/lib/locales/supported.d /config/.var/lib/locales/ \
 && mv /usr/lib/locale /config/.usr/lib/ \
 && ln -s /config/.var/lib/locales/supported.d /var/lib/locales/ \
 && ln -s /config/.usr/lib/locale /usr/lib/


# Setup the deluge user
RUN groupmod -o -g 8112 --new-name deluge debian-deluged \
 && usermod -o -u 8112 --login deluge --shell /bin/sh -d /config debian-deluged \
 && install -o deluge -g deluge -d /config /torrents /downloads


# Setup cmdline access to deluge-console
ENV TERM=linux TERMINFO=/etc/terminfo
RUN mkdir -p /root/.config && ln -s /config /root/.config/deluge


# Global config
ADD config/core/default+ /config/
ADD config/plugins/official/blocklist+ /config/
ADD config/plugins/official/notifications+ /config/
ADD config/plugins/official/web+ /config/
ADD config/plugins/third.party/autopriority+ /config/
ADD config/plugins/third.party/autoremoveplus+ /config/
ADD config/plugins/third.party/batchrenamer+ /config/
ADD config/plugins/third.party/labelplus+ /config/
ADD config/plugins/third.party/libtorrentconfig+ /config/
ADD config/plugins/third.party/myscheduler+ /config/
ADD config/plugins/third.party/webapi+ /config/
ADD config/users/deluge+ /config/


# Launch script
ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh


# Default container settings
VOLUME /config /torrents /downloads
EXPOSE 58846 8112
ENTRYPOINT ["/init","/entrypoint.sh","--config=/config"]


