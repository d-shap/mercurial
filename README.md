Mercurial web server
====================
Docker image for mercurial web server.

Container runs as non-root user.
This user owns mercurial process and owns mercurial repositories.

Web access is restricted with BASIC authentication for push requests, managed with Apache htpasswd.

To run container next volumes should be mapped
* repository root folder
* log folder
* htpasswd file to store user passwords

Installation
------------
Create user and group to own mercurial repositories and to run container
```
sudo useradd -r mercurial
```

Make **build** executable
```
sudo chmod u+x ./build
```

Execute **build**
```
sudo ./build mercurial mercurial
```

Create folder for mercurial repository
```
sudo mkdir /mercurial
```
```
sudo mkdir /mercurial/repositories
```

Create file to store user passwords
```
sudo touch /mercurial/users
```

Create folder for logs
```
sudo mkdir /var/log/mercurial
```

Grant permit to all files and folders
```
sudo chown -R mercurial:mercurial /mercurial
```
```
sudo chown mercurial:mercurial /var/log/mercurial
```

Copy **etc/init.d/hgweb** to **/etc/init.d** folder
```
sudo cp ./etc/init.d/hgweb /etc/init.d
```

Copy **usr/sbin/hgweb** to **/usr/sbin** folder
```
sudo cp ./usr/sbin/hgweb /usr/sbin
```

Copy **usr/bin/hgwebutil** to **/usr/bin** folder
```
sudo cp ./usr/bin/hgwebutil /usr/bin
```

Make all files executable
```
sudo chmod a+x /etc/init.d/hgweb
```
```
sudo chmod a+x /usr/sbin/hgweb
```
```
sudo chmod a+x /usr/bin/hgwebutil
```

Register service
```
sudo update-rc.d hgweb defaults
```

Management
----------
### Service management
```
sudo service hgweb (start|stop|status|restart)
```

### User management
```
sudo hgwebutil htpasswd [ -c ] [ -i ] [ -m | -B | -d | -s | -p ] [ -C cost ] [ -D ] [ -v ] passwdfile <username>
```
```
sudo hgwebutil htpasswd -b [ -c ] [ -m | -B | -d | -s | -p ] [ -C cost ] [ -D ] [ -v ] passwdfile <username> <password>
```

**htpasswd** command is passed to the running docker container with **hgwebutil** utility.

**username** and **password** should be replaced with real values.

**passwdfile** should be spelled literally.

### Mercurial repository management
```
sudo hgwebutil hg <command> <arguments>
```

**hg** command is passed to the running docker container with **hgwebutil** utility.

If command needs a path to the repository, this path is specified like this
```
PATH=/repopath/reponame
```

where **/repopath/reponame** is path started from **/mercurial/repositories**.

For example, to create a new repository, this command can be used
```
sudo hgwebutil hg init PATH=/newreponame
```

Apache mod_proxy configuration
-----------------------
Mercurial web server can be located with another web applications.
For example, bugzilla, artifactory, mercurial etc can be run as docker containers on the same host.
In this case apache can be used to redirect requests to different docker containers.

First, mod_proxy should be enabled
```
sudo a2enmod proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```

Configure proxy
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

Finally, restart apache service
```
sudo service apache2 restart
```
