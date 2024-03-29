FROM        debian:buster
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=969
ARG         hostgid=969

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER="mercurial" \
            RUN_GROUP="mercurial" \
            RUN_PORT=8009 \
            APACHE_RUN_USER="mercurial" \
            APACHE_RUN_GROUP="mercurial"

COPY        . /tmp

# Install common packages and configure environment
RUN         apt-get update && \
            apt-get upgrade --assume-yes && \
            apt-get install --assume-yes \
                apt-utils \
                build-essential \
                cron \
                curl \
                net-tools \
                procps \
                software-properties-common \
                vim \
                wget && \
            apt-get update && \
            apt-get install --assume-yes \
                php \
                tk && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Configure application folders
RUN         mkdir /mercurial && \
            mkdir /mercurial/repositories && \
            mkdir /mercurial/scripts && \
            mkdir /mercurial/logs && \
            mkdir /mercurial/backups && \
            chown -R $RUN_USER:$RUN_GROUP /mercurial

# Install Apache
RUN         apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                libapache2-mod-php && \
            apt-get clean && \
            sed -i 's|:80>|:'"$RUN_PORT"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            mv /tmp/mercurial.conf /etc/apache2/sites-available && \
            mv /tmp/mpasstool.conf /etc/apache2/sites-available && \
            sed -i 's|Listen 80|Listen '"$RUN_PORT"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod cgi expires headers mpm_prefork php7.3 rewrite && \
            a2ensite mercurial mpasstool && \
            rm -rf /var/log/apache2 && \
            ln -s /mercurial/logs /var/log/apache2 && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/apache2 && \
            mkdir -p /var/lock/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/lock/apache2 && \
            mkdir -p /var/run/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/apache2

# Configure mercurial
RUN         apt-get update && \
            apt-get install --assume-yes \
                mercurial && \
            apt-get clean && \
            mv /tmp/hgrc /etc/mercurial && \
            chmod 644 /etc/mercurial/hgrc && \
            mv /tmp/hgweb.config /mercurial && \
            chmod 644 /mercurial/hgweb.config && \
            sed 's|config = ".*"|config = "/mercurial/hgweb.config"|' /usr/share/doc/mercurial/examples/hgweb.cgi > /mercurial/hgweb.cgi && \
            chmod 755 /mercurial/hgweb.cgi

# Configure htpasstool
RUN         tar xzf /tmp/bin/htpasstool-v1.04.tgz -C /var/www/html && \
            rm /tmp/bin/htpasstool-v1.04.tgz && \
            rm -rf /tmp/bin && \
            mv /var/www/html/htpasstool-v1.04 /var/www/html/mpasstool && \
            mv /tmp/users /mercurial && \
            chmod 644 /mercurial/users && \
            cp /mercurial/users /mercurial/repositories && \
            ln -s /mercurial/repositories/users /var/www/html/mpasstool/.htpasswd && \
            echo "AuthType Basic" >> /var/www/html/mpasstool/.htaccess && \
            echo "AuthName \"Restricted Area\"" >> /var/www/html/mpasstool/.htaccess && \
            echo "AuthUserFile \"/var/www/html/mpasstool/.htpasswd\"" >> /var/www/html/mpasstool/.htaccess && \
            echo "require valid-user" >> /var/www/html/mpasstool/.htaccess && \
            chown -R -h $RUN_USER:$RUN_GROUP /var/www/html

# Configure helper scripts
RUN         mv /tmp/containerConfig /mercurial/scripts && \
            mv /tmp/containerDummy /mercurial/scripts && \
            mv /tmp/containerStartup /mercurial/scripts && \
            mv /tmp/containerShutdown /mercurial/scripts && \
            mv /tmp/webStartup /mercurial/scripts && \
            mv /tmp/webShutdown /mercurial/scripts && \
            mv /tmp/utilCreateRepositoryHgrc /mercurial/scripts && \
            mv /tmp/utilBackup /mercurial/scripts && \
            mv /tmp/utilRestore /mercurial/scripts && \
            chmod 755 /mercurial/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/mercurial/scripts/containerStartup"]
