---
layout: post
title: "Building Openresty on Ubuntu"
date: 2013-03-19 16:14
comments: true
categories: 
- nginx
- ubuntu
- Lua
---

A question that comes up frequently on the
[openresty mailing list](http://groups.google.com/group/openresty-en)
is an easy way to install on Ubuntu. While I'm not ready to start
maintaining a package repo, I figured I could share how I build and
package openresty for my own server.  

<!--more-->

I'll walk through this manually, but you could create a shell script
to do this easily.

First, install some prerequisites:

    sudo apt-get -y install make ruby1.9.1 ruby1.9.1-dev git-core \
    libpcre3-dev libxslt1-dev libgd2-xpm-dev libgeoip-dev unzip zip build-essential
    
    sudo gem install fpm

Now, get openresty. I generally stick with the stable versions unless
I'm testing something:

    wget http://openresty.org/download/ngx_openresty-1.2.6.6.tar.gz
    tar -zxvf ngx_openresty-1.2.6.6.tar.gz
    cd ngx_openresty-1.2.6.6
   
 Now configure and build. This will configure openresty like the
 Ubuntu nginx package (more or less):
 
     ./configure \
    --with-luajit \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-client-body-temp-path=/var/lib/nginx/body \
    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
    --http-log-path=/var/log/nginx/access.log \
    --http-proxy-temp-path=/var/lib/nginx/proxy \
    --http-scgi-temp-path=/var/lib/nginx/scgi \
    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
    --lock-path=/var/lock/nginx.lock \
    --pid-path=/var/run/nginx.pid \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_geoip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-http_ssl_module \
    --with-http_sub_module \
    --with-http_xslt_module \
    --with-ipv6 \
    --with-sha1=/usr/include/openssl \
    --with-md5=/usr/include/openssl \
    --with-mail \
    --with-mail_ssl_module \
    --with-http_stub_status_module \
    --with-http_secure_link_module \
    --with-http_sub_module 
    
    make
    
 Rather than installing, we will use
 [fpm](https://github.com/jordansissel/fpm) to build a package:
 
    INSTALL=/tmp/openresty
    make install DESTDIR=$INSTALL
    mkdir -p $INSTALL/var/lib/nginx
    install -m 0555 -D nginx.init $INSTALL/etc/init.d/nginx
    install -m 0555 -D nginx.logrotate $INSTALL/etc/logrotate.d/nginx

    fpm -s dir -t deb -n nginx -v 1.2.6 --iteration 1 -C $INSTALL \
    --description "openresty 1.2.6.6" \
    -d libxslt1.1 \
    -d libgd2-xpm \
    -d libgeoip1 \
    -d libpcre3 \
    --config-files etc/nginx/fastcgi.conf.default \
    --config-files /etc/nginx/win-utf \
    --config-files /etc/nginx/conf.d/default.conf \
    --config-files /etc/nginx/fastcgi_params \
    --config-files /etc/nginx/nginx.conf \
    --config-files /etc/nginx/koi-win \
    --config-files /etc/nginx/nginx.conf.default \
    --config-files /etc/nginx/mime.types.default \
    --config-files /etc/nginx/koi-utf \
    --config-files /etc/nginx/uwsgi_params \
    --config-files /etc/nginx/uwsgi_params.default \
    --config-files /etc/nginx/sites-available/default \
    --config-files /etc/nginx/fastcgi_params.default \
    --config-files /etc/nginx/mime.types \
    --config-files /etc/nginx/scgi_params.default \
    --config-files /etc/nginx/scgi_params \
    --config-files /etc/nginx/fastcgi.conf \
    etc usr var
    
The files `nginx.init` and `nginx.logrotate` can be found below.

You should now have a package in your current directory.

{% gist 5199830 nginx.init %}

{% gist 5199830 nginx.logrotate %}

