FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=969
ARG         hostgid=969
ARG         guestport=8008
ARG         guestappname=mercurial

ENV         DEBIAN_FRONTEND noninteractive
ENV         APACHE_RUN_USER mercurial
ENV         APACHE_RUN_GROUP mercurial
ENV         GUEST_APP_NAME $guestappname

COPY        . /tmp

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                mercurial \
                tk \
                vim && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid mercurial && \
            useradd -u $hostuid -g $hostgid -M mercurial

# Create files and folders for mercurial
RUN         mkdir /mercurial && \
            mkdir /mercurial/repositories && \
            mkdir /mercurial/scripts && \
            touch /mercurial/users && \
            mkdir /mercurial/logs && \
            mkdir /mercurial/backups && \
            chown -R mercurial:mercurial /mercurial

# Configure apache
RUN         sed -i 's|:80>|:'"$guestport"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            mv /tmp/mercurial.conf /etc/apache2/sites-available && \
            sed -i 's|/guestappname|/'"$guestappname"'|' /etc/apache2/sites-available/mercurial.conf && \
            sed -i 's|Listen 80|Listen '"$guestport"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi && \
            a2ensite mercurial && \
            rm -rf /var/log/apache2 && \
            ln -s /mercurial/logs /var/log/apache2 && \
            chown -R mercurial:mercurial /var/log/apache2 && \
            chown -R mercurial:mercurial /var/lock/apache2 && \
            chown -R mercurial:mercurial /var/run/apache2

# Configure mercurial
RUN         mv /tmp/hgrc /etc/mercurial && \
            sed -i 's|/guestappname|/'"$guestappname"'|' /etc/mercurial/hgrc && \
            mv /tmp/hgweb.config /mercurial && \
            chown mercurial:mercurial /mercurial/hgweb.config && \
            sed 's|config = ".*"|config = "/mercurial/hgweb.config"|' /usr/share/doc/mercurial/examples/hgweb.cgi > /mercurial/hgweb.cgi && \
            chmod a+x /mercurial/hgweb.cgi

# Configure helper scripts
RUN         mv /tmp/startup /mercurial/scripts && \
            chmod a+x /mercurial/scripts/startup && \
            mv /tmp/shutdown /mercurial/scripts && \
            chmod a+x /mercurial/scripts/shutdown && \
            mv /tmp/createrepositoryhgrc /mercurial/scripts && \
            chmod a+x /mercurial/scripts/createrepositoryhgrc && \
            mv /tmp/backup /mercurial/scripts && \
            chmod a+x /mercurial/scripts/backup && \
            mv /tmp/restore /mercurial/scripts && \
            chmod a+x /mercurial/scripts/restore

USER        mercurial

EXPOSE      $guestport

ENTRYPOINT  ["/mercurial/scripts/startup"]
