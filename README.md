Apache+PHP build pack
========================

This is a build pack bundling PHP and Apache for Heroku apps.

Configuration
-------------

The config files are bundled with the build pack itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------

    export APACHE_VERSION=2.4.12
    export PHP_VERSION=5.6.5
    
    # apache
    ## dependencies
    curl -LO http://mirror.csclub.uwaterloo.ca/apache/apr/apr-1.5.1.tar.bz2
    tar xjf apr-*.tar.bz2 && rm apr-*.tar.bz2
    curl -LO http://mirror.csclub.uwaterloo.ca/apache/apr/apr-util-1.5.4.tar.bz2
    tar xjf apr-util-*.tar.bz2 && rm apr-util-*.tar.bz2
    curl -LO ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.bz2
    tar xjf pcre-*.tar.bz2 && rm pcre-*.tar.bz2
    cd pcre-*
    ./configure --prefix=/app/usr/local/pcre && make && make install
    cd ..
    
    ## httpd
    curl -LO http://mirror.its.dal.ca/apache//httpd/httpd-${APACHE_VERSION}.tar.bz2
    tar xjf httpd-*.tar.bz2 && rm httpd-*.tar.bz2
    mv apr-util-* httpd-${APACHE_VERSION}/srclib/apr-util
    mv apr-* httpd-${APACHE_VERSION}/srclib/apr
    cd httpd-${APACHE_VERSION}
    ./configure --with-mpm=prefork --prefix=/app/apache --with-included-apr --with-pcre=/app/usr/local/pcre --enable-rewrite --enable-expires --enable-deflate --enable-headers && make && make install
    cd ..
    
    # php
    curl -Lo php.tar.bz2 http://ca1.php.net/get/php-${PHP_VERSION}.tar.bz2/from/this/mirror
    tar xjf php.tar.bz2 && rm php.tar.bz2
    cd php-${PHP_VERSION}/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl && make && make install
    cd ..
    
    # pear
    /app/php/bin/pear config-set php_dir /app/php
    /app/php/bin/pear install Mail Net_SMTP
    /app/php/bin/pecl uninstall apc
    /app/php/bin/pecl install apc
    cp /app/php/lib/php/extensions/*/apc.so /app/php/ext/
    
    # package
    cd /app/
    cp -r /app/usr/local/pcre/lib/* apache/lib/
    echo "${APACHE_VERSION}" > apache/VERSION
    tar cjf apache-${APACHE_VERSION}.tar.bz2 apache
    echo "${PHP_VERSION}" > php/VERSION
    tar cjf php-${PHP_VERSION}.tar.bz2 php

    # Upload the packages
    curl -u gb -F upfile=@apache-${APACHE_VERSION}.tar.bz2 -F go=Send+File http://pub.pommepause.com/
    curl -u gb -F upfile=@php-${PHP_VERSION}.tar.bz2 -F go=Send+File http://pub.pommepause.com/
    # or scp *.bz2 ...


Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
