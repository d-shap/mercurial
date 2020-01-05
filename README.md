# Mercurial web server
Docker image for mercurial web server.

Container runs as non-root user.
This user owns mercurial process and owns mercurial repositories.

To run container next volumes should be mapped:
* repository root folder
* log folder
* backups folder

## Installation
### Installation from docker image
1. Pull docker image.

2. Create user and group to own mercurial repositories and to run docker container:
    ```
    sudo groupadd -g 969 mercurial
    ```
    ```
    useradd -u 969 -g 969 -M mercurial
    ```

3. Proceed to configuration.

### Installation from source
1. Pull project sources from version control system.

2. Create user and group to own mercurial repositories and to run docker container:
    ```
    sudo useradd -r mercurial
    ```

3. Make **build** executable:
    ```
    sudo chmod u+x ./build
    ```

4. Execute **build**:
    ```
    sudo ./build mercurial
    ```

Proceed to configuration.

### Configuration
1. Create folders for mercurial repository:
    ```
    sudo mkdir /mercurial
    ```
    ```
    sudo mkdir /mercurial/repositories
    ```

2. Create folder for logs:
    ```
    sudo mkdir /var/log/mercurial
    ```

3. Create folder for backups:
    ```
    sudo mkdir /var/backups/mercurial
    ```

4. Grant permit to all files and folders:
    ```
    sudo chown -R mercurial:mercurial /mercurial
    ```
    ```
    sudo chown mercurial:mercurial /var/log/mercurial
    ```
    ```
    sudo chown mercurial:mercurial /var/backups/mercurial
    ```

5. Copy **./etc/init.d/mercurial** to **/etc/init.d** folder:
    ```
    sudo cp ./etc/init.d/mercurial /etc/init.d
    ```

6. Copy **./usr/sbin/mercurial** to **/usr/sbin** folder:
    ```
    sudo cp ./usr/sbin/mercurial /usr/sbin
    ```

7. Copy **./usr/bin/mutil** to **/usr/bin** folder:
    ```
    sudo cp ./usr/bin/mutil /usr/bin
    ```

8. Make all files executable:
    ```
    sudo chmod a+x /etc/init.d/mercurial
    ```
    ```
    sudo chmod a+x /usr/sbin/mercurial
    ```
    ```
    sudo chmod a+x /usr/bin/mutil
    ```

9. Register service:
    ```
    sudo update-rc.d mercurial defaults
    ```

10. Start mercurial service:
    ```
    sudo service mercurial start
    ```

## Client configuration
1. Create file **~/.hgrc** (Linux) or **%systemdrive%%homepath%\mercurial.ini** (Windows).

2. Specify user name and e-mail in this file:
    ```
    [ui]
    username = First Last <e-mail@example.com>
    ```

## Management
### Service management
```
sudo service mercurial (start|stop|status|restart)
```

### Repository management
```
sudo mutil hg <command> <arguments>
```

**hg** command is passed to the running docker container with **mutil** utility.

If command needs a path to the repository, this path is specified like this:
```
PATH=/repopath/reponame
```

### Create repository backup
```
sudo mutil backup <filename>
```

Backup file **/var/backups/mercurial/&lt;filename&gt;.tar.gz** will be created.

### Restore repository backup
```
sudo mutil restore <filename>
```

### Command line (bash)
```
sudo mutil bash
```

## Apache mod_proxy configuration
Mercurial web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

1. Enable apache mod_proxy:
    ```
    sudo a2enmod proxy proxy_ajp proxy_html proxy_http rewrite deflate headers proxy_balancer proxy_connect
    ```

2. Configure proxy:
    ```
    <VirtualHost *:80>
        ...
        ProxyPreserveHost On
        <Proxy *>
            Order allow,deny
            Allow from all
        </Proxy>
        ...
    </VirtualHost>
    ```

3. Copy **./etc/apache2/sites-available/mercurial.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/mercurial.conf /etc/apache2/sites-available
    ```

4. Copy **./etc/apache2/sites-available/mpasstool.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/mpasstool.conf /etc/apache2/sites-available
    ```

5. Enable apache mercurial site:
    ```
    sudo a2ensite mercurial mpasstool
    ```

6. Restart apache service:
    ```
    sudo service apache2 restart
    ```

## HOW TO
### How to create a new user
```
sudo mutil htpasswd passwdfile <username>
```

### How to create a new repository
```
sudo mutil hg init PATH=/newreponame
```
```
sudo mutil hg init PATH=/newrepopath/newreponame
```

### How to change repository description
Edit **contact** and **description** fields in **/mercurial/repositories/repopath/reponame/.hg/hgrc** file:
```
[web]
...
contact         = Repository contact person
description     = Repository description
...
```

### How to restrict access to repository
Edit **allow_read** and **allow_push** fields in **/mercurial/repositories/repopath/reponame/.hg/hgrc** file:
```
[web]
...
allow_read      = *
allow_push      = username
...
```

### How to change default theme
Specify **WEB_STYLE** environment variable in **/usr/sbin/mercurial** file:
```
docker run ... -e WEB_STYLE="gitweb" ...
```

Available themes are:
* paper (default theme)
* coal
* spartan
* monoblue
* gitweb

### How to disable multi-level directory hierarchy
Specify **WEB_COLLAPSE** environment variable in **/usr/sbin/mercurial** file:
```
docker run ... -e WEB_COLLAPSE="false" ...
```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/mutil backup <filename>"; echo ""; } | sudo crontab -
```

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
