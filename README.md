## Nginx Installation
Create the REPO file
---
`vi /etc/yum.repos.d/nginx.repo`
---
`[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/{OS}/{VER}/$basearch/
gpgcheck=0
enabled=1`
---

Install the Nginx packages.
---
`yum --nogpgcheck -y install nginx nginx-module-geoip nginx-module-image-filter`
---


## Configuration files##
`cd /etc/nginx`
---
`mv nginx.conf nginx.conf.orig
git init
git remote add ehorigin https://github.com/mikeweb85/nginx-conf.git
git pull ehorigin master
git remote rm ehorigin`
---
`find . -type f -exec sed -i "s|{HOSTNAME}|$(hostname -s)|g" {} \;
find . -type f -exec sed -i "s|{NODE-ENV}|PROD|g" {} \;
find . -type f -exec sed -i "s|{NODE-NAME}|WW1|g" {} \;
find . -type f -exec sed -i "s|{SELF-IP}|$(ifconfig  | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk '{ print $1}')|g" {} \;`
---

### GeoIP Data ###
The module is disabled by default, but having these files ready can make it easy to simply enable the file when needed.
---
`cd /etc/nginx/geoip
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
tar -xf GeoIP.dat.gz
tar -xf GeoLiteCity.dat.gz`
---


## Additional Modules ##
Nginx dynamic modules are version specific. Get the installed version then download its source.
---
`cd /root/
nginx -v`
---
`yum --nogpgcheck -y install cmake gcc gcc-c++ glib2-devel zlib-devel pcre-devel openssl-devel
wget http://nginx.org/download/nginx-{VERSION}.tar.gz
tar -xf nginx-{VERSION}.tar.gz`
---
`mkdir nginx-modules
cd nginx-modules`
---

### Upload Progress ###
Docs: https://github.com/masterzen/nginx-upload-progress-module
Releases: https://github.com/masterzen/nginx-upload-progress-module/releases
---
`wget -Onginx-upload-progress-module-{RELEASE}.tar.gz  https://github.com/masterzen/nginx-upload-progress-module/archive/v{RELEASE}.tar.gz
tar -xf nginx-upload-progress-module-{RELEASE}.tar.gz`
---

### Virtual Host Status ###
Docs: https://github.com/vozlt/nginx-module-vts
Releases: https://github.com/vozlt/nginx-module-vts/releases
---
`wget -Onginx-vts-module-{RELEASE}.tar.gz  https://github.com/vozlt/nginx-module-vts/archive/v{RELEASE}.tar.gz
tar -xf nginx-vts-module-{RELEASE}.tar.gz`
---

### Headers More ###
Docs: https://github.com/openresty/headers-more-nginx-module
Releases: https://github.com/openresty/headers-more-nginx-module/releases
---
`wget -Onginx-headers-more-module-{RELEASE}.tar.gz  https://github.com/openresty/headers-more-nginx-module/archive/v{RELEASE}.tar.gz
tar -xf nginx-headers-more-module-module-{RELEASE}.tar.gz`
---

## Compiling the Modules ##
Each module will extract itself to a separate directory. For every module you will need to include the following template for the final configure command:
---
``--add-dynamic-module=/root/nginx-modules/{MODULE-DIR}``
---

Now we take what we have done so far and begin the build
---
`cd /root/nginx-{VERSION}
./configure --with-compat {ADD-DYNAMIC-MODULES}
make modules`
---

If the compilation was successful:
---
`cp -r /root/nginx-{VERSION}/objs/*.so /etc/nginx/modules/`
---
