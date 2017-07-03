FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=987
ARG         hostgid=987
ARG         guestport=8008
ARG         guestappname=mercurial

ENV         DEBIAN_FRONTEND noninteractive
ENV         APACHE_RUN_USER mercurial
ENV         APACHE_RUN_GROUP mercurial

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes apache2 mercurial

# Create user and group
RUN         groupadd -g $hostgid mercurial && \
            useradd -u $hostuid -g $hostgid -M mercurial

# Create folder for mercurial
RUN         mkdir /mercurial && \
            chown mercurial:mercurial /mercurial

COPY        hgrc /etc/mercurial
COPY        hgweb.config /mercurial
COPY        mercurial.conf /etc/apache2/sites-available
COPY        startup /mercurial
COPY        createrepositoryhgrc /mercurial

# Configure mercurial
RUN         sed -i 's|/guestappname|/'"$guestappname"'|' /etc/mercurial/hgrc && \
            chown mercurial:mercurial /mercurial/hgweb.config && \
            sed 's|config = ".*"|config = "/mercurial/hgweb.config"|' /usr/share/doc/mercurial/examples/hgweb.cgi > /mercurial/hgweb.cgi && \
            chmod a+x /mercurial/hgweb.cgi

# Configure apache
RUN         sed -i 's|:80>|:'"$guestport"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|/guestappname|/'"$guestappname"'|' /etc/apache2/sites-available/mercurial.conf && \
            sed -i 's|Listen 80|Listen '"$guestport"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi && \
            a2ensite mercurial && \
            chown -R mercurial:mercurial /var/run/apache2 && \
            chown -R mercurial:mercurial /var/log/apache2

# Grant permits for helper scripts
RUN         chmod a+x /mercurial/startup && \
            chmod a+x /mercurial/createrepositoryhgrc

USER        mercurial

EXPOSE      $guestport

ENTRYPOINT  ["/mercurial/startup"]
