## Nginx Installation
Create the REPO file with: `vi /etc/yum.repos.d/nginx.repo`
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/{OS}/{VER}/$basearch/
gpgcheck=0
enabled=1
```

Install the Nginx packages.
```
yum --nogpgcheck -y install nginx nginx-module-geoip nginx-module-image-filter
```


## Configuration files
Move into the Nginx directory: `cd /etc/nginx`. Before replacing the global variables, it is important to note the **{NODE}** and **{ENV}** variables. **{NODE}** is simply a generic *ww1/ww2/etc* naming convention to assist when troubleshooting in load balanced scenarios. **{ENV}** is the primary environment of the web server, usually noted as *PROD*, *STAGE*, *QA*, *DEV*, etc.
```
mv nginx.conf nginx.conf.orig
git init
git remote add ehorigin https://github.com/mikeweb85/nginx-conf.git
git pull ehorigin master
git remote rm ehorigin
rm -Rf ./.git*

find . -type f -exec sed -i "s|{NODE-ENV}|{ENV}|g" {} \;
find . -type f -exec sed -i "s|{NODE-NAME}|{NODE}|g" {} \;
find . -type f -exec sed -i "s|{HOSTNAME}|$(hostname -s)|g" {} \;
find . -type f -exec sed -i "s|{SELF-IP}|$(hostname -i)|g" {} \;
```
Create cache directories
```
mkdir -p /var/nginx/cache/{client_body,fastcgi,fastcgi_tmp,proxy,proxy_tmp}
```

Once the files have been updated, enable your first virtual server.
```
cd /etc/nginx/sites-enabled
ln -s ../sites-available/localhost.conf
```

### GeoIP Data
The module is disabled by default, but having these files ready can make it easy to simply enable the file when needed.
```
cd /etc/nginx/geoip
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
gunzip GeoIP.dat.gz
gunzip GeoLiteCity.dat.gz
```


## Additional Modules
Nginx dynamic modules are version specific. Get the installed version then download its source.
```
mkdir -p /opt/nginx/nginx-modules
cd /opt/nginx/
nginx -v

yum --nogpgcheck -y install cmake gcc gcc-c++ glib2-devel zlib-devel pcre-devel openssl-devel
wget http://nginx.org/download/nginx-{VERSION}.tar.gz
tar -xf nginx-{VERSION}.tar.gz

cd nginx-modules
```

### Upload Progress
Docs: https://github.com/masterzen/nginx-upload-progress-module

Releases: https://github.com/masterzen/nginx-upload-progress-module/releases
```
wget -Onginx-upload-progress-module-{RELEASE}.tar.gz  https://github.com/masterzen/nginx-upload-progress-module/archive/v{RELEASE}.tar.gz
tar -xf nginx-upload-progress-module-{RELEASE}.tar.gz
```

### Virtual Host Status
Docs: https://github.com/vozlt/nginx-module-vts

Releases: https://github.com/vozlt/nginx-module-vts/releases
```
wget -Onginx-vts-module-{RELEASE}.tar.gz  https://github.com/vozlt/nginx-module-vts/archive/v{RELEASE}.tar.gz
tar -xf nginx-vts-module-{RELEASE}.tar.gz
```

### Headers More
Docs: https://github.com/openresty/headers-more-nginx-module

Releases: https://github.com/openresty/headers-more-nginx-module/releases
```
wget -Onginx-headers-more-module-{RELEASE}.tar.gz  https://github.com/openresty/headers-more-nginx-module/archive/v{RELEASE}.tar.gz
tar -xf nginx-headers-more-module-{RELEASE}.tar.gz
```


## Compiling the Modules
Each module will extract itself to a separate directory. For every module you will need to include the following template for the final configure command:
```
--add-dynamic-module=/opt/nginx/nginx-modules/{MODULE-DIR}
```

Now we take what we have done so far and begin the build
```
cd /opt/nginx/nginx-{VERSION}
./configure --with-compat {ADD-DYNAMIC-MODULES}
make modules
```

If the compilation was successful:
```
cp -r /opt/nginx/nginx-{VERSION}/objs/*.so /etc/nginx/modules/
```


## Adding Virtual Servers
Most of the sites will be based on existing apps and have corresponding examples. The examples are written with the *www* prefix included and the document root directory assumed to be `/var/www/clients/{TLD}/html` where **{TLD}** is the TLD (domain) to be configured.

Create the document root and stage site files (if applicable)
```
mkdir -p /var/www/clients/{TLD}/html
cd /var/www/clients/{TLD}
chown -R nginx:nginx .
find . -type d -exec chmod 755 {} \; && find . -type f -exec chmod 644 {} \;
```
Create the virtual server configuration file.
```
cd /etc/nginx/sites-available
cp ../examples/{APP}/{APP}.conf ./{TLD}.conf
sed -i "s|{DOMAIN}|{TLD}|g" {TLD}.conf
```
Make any final configuration then enable the site to be loaded into Nginx.
```
cd /etc/nginx/sites-enabled
ln -s ../sites-available/{TLD}.conf
```
Verify your changes with `nginx -t`. Restart or reload Nginx with `service nginx restart` or `service nginx reload`
