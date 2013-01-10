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

    # apache
    ## dependencies
    curl -O http://mirror.csclub.uwaterloo.ca/apache/apr/apr-1.4.6.tar.bz2
    tar xjf apr-1.4.6.tar.bz2 && rm apr-1.4.6.tar.bz2
    curl -O http://mirror.csclub.uwaterloo.ca/apache/apr/apr-util-1.5.1.tar.bz2
    tar xjf apr-util-1.5.1.tar.bz2 && rm apr-util-1.5.1.tar.bz2
    curl -Lo pcre.tar.bz2 ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.31.tar.bz2
    tar xjf pcre.tar.bz2 && rm pcre.tar.bz2
    cd pcre-*
    ./configure --prefix=/app/usr/local/pcre
    make && make install
    cd ..
    
    ## httpd
    curl -Lo httpd.tar.bz2 http://apache.mirror.nexicom.net/httpd/httpd-2.4.3.tar.bz2
    tar xjf httpd.tar.bz2 && rm httpd.tar.bz2
    mv apr-1.4.6 httpd-2.4.3/srclib/apr
    mv apr-util-1.5.1 httpd-2.4.3/srclib/apr-util
    cd httpd-*
    ./configure --with-mpm=prefork --prefix=/app/apache --with-included-apr --with-pcre=/app/usr/local/pcre --enable-rewrite --enable-expires --enable-deflate --enable-headers
    make && make install
    cd ..
    
    # php
    curl -Lo php.tar.gz http://ca1.php.net/get/php-5.3.18.tar.gz/from/us1.php.net/mirror
    tar xzf php.tar.gz && rm php.tar.gz
    cd php-*/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-mysqli --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl
    make && make install
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
    echo '2.4.3' > apache/VERSION
    tar cjf apache-2.4.3.tar.bz2 apache
    echo '5.3.18' > php/VERSION
    tar cjf php-5.3.18.tar.bz2 php
    scp *.bz2 ...


Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
