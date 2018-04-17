Mercurial web server
====================
Docker image for mercurial web server.

Container runs as non-root user.
This user owns mercurial process and owns mercurial repositories.

To run container next volumes should be mapped:
* repository root folder
* htpasswd file to store hashes of user passwords
* log folder
* backups folder

Installation
------------
Create user and group to own mercurial repositories and to run docker container:
```
sudo useradd -r mercurial
```

Make **build** executable:
```
sudo chmod u+x ./build
```

Execute **build**:
```
sudo ./build mercurial
```

Create folders for mercurial repository:
```
sudo mkdir /mercurial
```
```
sudo mkdir /mercurial/repositories
```

Create file to store user passwords:
```
sudo touch /mercurial/users
```

Create folder for logs:
```
sudo mkdir /var/log/mercurial
```

Create folder for backups:
```
sudo mkdir /var/backups/mercurial
```

Grant permit to all files and folders:
```
sudo chown -R mercurial:mercurial /mercurial
```
```
sudo chown mercurial:mercurial /var/log/mercurial
```
```
sudo chown mercurial:mercurial /var/backups/mercurial
```

Copy **etc/init.d/mercurial** to **/etc/init.d** folder:
```
sudo cp ./etc/init.d/mercurial /etc/init.d
```

Copy **usr/sbin/mercurial** to **/usr/sbin** folder:
```
sudo cp ./usr/sbin/mercurial /usr/sbin
```

Copy **usr/bin/mutil** to **/usr/bin** folder:
```
sudo cp ./usr/bin/mutil /usr/bin
```

Make all files executable:
```
sudo chmod a+x /etc/init.d/mercurial
```
```
sudo chmod a+x /usr/sbin/mercurial
```
```
sudo chmod a+x /usr/bin/mutil
```

Register service:
```
sudo update-rc.d mercurial defaults
```

Start mercurial service:
```
sudo service mercurial start
```

Management
----------
### Service management
```
sudo service mercurial (start|stop|status|restart)
```

### User management
```
sudo mutil htpasswd <arguments> passwdfile <username>
```
```
sudo mutil htpasswd -b <arguments> passwdfile <username> <password>
```

**htpasswd** command is passed to the running docker container with **mutil** utility.

**username** and **password** should be replaced with real values.

**passwdfile** should be spelled literally.


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
sudo mutil backup
```

The backup contains **/mercurial/repositories** folder and **/mercurial/users** file.

### Restore repository backup
```
sudo mutil restore
```

After backup is restored, you should restore file with passwords:
```
sudo service mercurial stop
```
```
sudo mv /mercurial/repositories/users /mercurial
```
```
sudo service mercurial start
```

### Command line (bash)
```
sudo mutil bash
```

Apache mod_proxy configuration
------------------------------
Mercurial web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

First, mod_proxy should be enabled:
```
sudo a2enmod proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```

Then configure proxy:
```
<VirtualHost *:80>

...

ProxyPreserveHost On
<Proxy *>
    Order allow,deny
    Allow from all
</Proxy>

...

ProxyPass /mercurial http://localhost:8008/mercurial
ProxyPassReverse /mercurial http://localhost:8008/mercurial

...

</VirtualHost>
```

Finally, restart apache service:
```
sudo service apache2 restart
```

HOW TO
------
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
Edit **contact** and **description** fields in **hgrc** file:
```
sudo vi /mercurial/repositories/repopath/reponame/.hg/hgrc
```

### How to restrict access to repository
Edit **allow_read** and **allow_push** fields in **hgrc** file:
```
sudo vi /mercurial/repositories/repopath/reponame/.hg/hgrc
```

### How to change default theme
```
sudo vi /usr/sbin/mercurial
```

Add **WEB_STYLE** environment variable to the docker run command:
```
docker run ... -e WEB_STYLE=gitweb ...
```

Available themes are:
* paper (default theme)
* coal
* spartan
* monoblue
* gitweb

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "10 2 * * * /usr/bin/mutil backup"; echo ""; } | sudo crontab -
```
