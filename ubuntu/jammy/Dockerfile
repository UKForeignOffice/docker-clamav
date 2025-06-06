FROM ubuntu:22.04
LABEL maintainer="Markus Kosmal <code@m-ko.de>"

# initial install of av daemon
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y -qq \
        clamav-daemon \
        clamav-freshclam \
        libclamunrar9 \
        ca-certificates \
        netcat-openbsd \
        wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# initial update of av databases
# disabled - see https://github.com/mko-x/docker-clamav#prefer-alpine-idb-amd64
#RUN { wget -O /var/lib/clamav/main.cvd http://database.clamav.net/main.cvd && \
#    wget -O /var/lib/clamav/daily.cvd http://database.clamav.net/daily.cvd && \
#    wget -O /var/lib/clamav/bytecode.cvd http://database.clamav.net/bytecode.cvd && \
#    chown clamav:clamav /var/lib/clamav/*.cvd } || { echo DB Initialisation failed }

# permission juggling
RUN mkdir /var/run/clamav && \
    chown clamav:clamav /var/run/clamav && \
    chmod 750 /var/run/clamav

# av configuration update
RUN sed -i 's/^Foreground .*$/Foreground true/g' /etc/clamav/clamd.conf && \
    echo "TCPSocket 3310" >> /etc/clamav/clamd.conf && \
    if [ -n "$HTTPProxyServer" ]; then echo "HTTPProxyServer $HTTPProxyServer" >> /etc/clamav/freshclam.conf; fi && \
    if [ -n "$HTTPProxyPort"   ]; then echo "HTTPProxyPort $HTTPProxyPort" >> /etc/clamav/freshclam.conf; fi && \
    if [ -n "$DatabaseMirror"  ]; then echo "DatabaseMirror $DatabaseMirror" >> /etc/clamav/freshclam.conf; fi && \
    if [ -n "$DatabaseMirror"  ]; then echo "ScriptedUpdates off" >> /etc/clamav/freshclam.conf; fi && \
    sed -i 's/^Foreground .*$/Foreground true/g' /etc/clamav/freshclam.conf


# env based configs - will be called by bootstrap.sh
COPY envconfig.sh /
COPY check.sh /
COPY bootstrap.sh /

# port provision
EXPOSE 3310

RUN chown clamav:clamav bootstrap.sh check.sh envconfig.sh /etc/clamav /etc/clamav/clamd.conf /etc/clamav/freshclam.conf && \
    chmod u+x bootstrap.sh check.sh envconfig.sh    

USER clamav

CMD ["/bootstrap.sh"]
