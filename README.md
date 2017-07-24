Mercurial web server
====================
Docker image for mercurial web server.

Container runs as non-root user.
This user owns mercurial process and owns mercurial repositories.

Web access is restricted with BASIC authentication for push requests, managed with Apache htpasswd.

To run container next volumes should be mapped
* repository root folder
* htpasswd file to store hashes of user passwords
* log folder

Installation
------------
Create user and group to own mercurial repositories and to run docker container
```
sudo useradd -r mercurial
```

Make **build** executable
```
sudo chmod u+x ./build
```

Execute **build**
```
sudo ./build mercurial
```

Create folders for mercurial repository
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

Copy **etc/init.d/mercurial** to **/etc/init.d** folder
```
sudo cp ./etc/init.d/mercurial /etc/init.d
```

Copy **usr/sbin/mercurial** to **/usr/sbin** folder
```
sudo cp ./usr/sbin/mercurial /usr/sbin
```

Copy **usr/bin/mutil** to **/usr/bin** folder
```
sudo cp ./usr/bin/mutil /usr/bin
```

Make all files executable
```
sudo chmod a+x /etc/init.d/mercurial
```
```
sudo chmod a+x /usr/sbin/mercurial
```
```
sudo chmod a+x /usr/bin/mutil
```

Register service
```
sudo update-rc.d mercurial defaults
```

Start mercurial service
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
sudo mutil htpasswd [ -c ] [ -i ] [ -m | -B | -d | -s | -p ] [ -C cost ] [ -D ] [ -v ] passwdfile <username>
```
```
sudo mutil htpasswd -b [ -c ] [ -m | -B | -d | -s | -p ] [ -C cost ] [ -D ] [ -v ] passwdfile <username> <password>
```

**htpasswd** command is passed to the running docker container with **mutil** utility.

**username** and **password** should be replaced with real values.

**passwdfile** should be spelled literally.


### Repository management
```
sudo mutil hg <command> <arguments>
```

**hg** command is passed to the running docker container with **mutil** utility.

If command needs a path to the repository, this path is specified like this
```
PATH=/repopath/reponame
```

where **/repopath/reponame** is path started from **/mercurial/repositories**.

For example, to create a new repository, this command can be used
```
sudo mutil hg init PATH=/newreponame
```

Apache mod_proxy configuration
------------------------------
Mercurial web server can be located with another web applications.
For example, bugzilla, artifactory, mercurial etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

First, mod_proxy should be enabled
```
sudo a2enmod proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```

Then configure proxy
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

Customization
-------------
### Change default theme
Add **WEB_STYLE** environment variable to the docker run command
```
docker run ... -e WEB_STYLE=gitweb ...
```

Available themes are:
* paper (default theme)
* coal
* spartan
* monoblue
* gitweb
