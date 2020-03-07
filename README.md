# FreeBSD Apache Sparry
Sparry is a acronym for "SPawn Apache viRtualhosts eveRYwhere" (sorry, couldn't come up with something better ;). Sparry is a easy to use management tool for apache VirtualHost configurations on FreeBSD. With Sparry most people should be able to create apache VirtualHost configurations that are simple and straight forward, but also reasonably secure.

*Don't use Sparry yet: it's still under development!*

## Features
* **Easy to use**: Sparry is made for (and with!) people that normally would get discouraged if they had to create VirtualHost files on servers.
* **Automated TLS certificates** with Let's Encrypt official certbot client (**under development**).
* **Automated HTTP Security Headers** based on your use-case (different profiles available).
* **Automated Error logging** based on your use-case (different options available).
* **Made for FreeBSD** and works on basic shell (so no advanced shells like bash or zsh required).

## How does it work
Sparry asks some questions and will generate a VirtualHost file and configure apache based on the answers given. Nothing too special. There is one VirtualHost configuration file for every domain (with or without the www. subdomain and for both http and https).

## Limitations
Sparry is really meant for basic tasks and is not for you if you require stuff like other subdomains than `www`, multiple domains in one file or the more complex Apache stuff.

## Examples
### Runthrough
```
root@nozel:~ # sparry --add-webconfig

Sparry will guide you through the creation of a new apache
configuration file now. Please answer the following questions:

(1) Did you configure and propagate relevant DNS records? [yes/no]: yes
(2) Enter domain name (e.g. domain.tld): nozel.org
    Performing DNS lookup for nozel.org
    Domain nozel.org OK
(3) Select (sub)domain layout:
    1 nozel.org
    2 www.nozel.org
    3 nozel.org and www.nozel.org
    (1-3): 3
(4) DocumentRoot (full path or empty for FreeBSD default):
    :
    default path /usr/local/www will be used
(5) Add TLS certificate?
    1 TLS certificate with RSA key size of 2048 bits (default)
    2 TLS certificate with RSA key size of 4096 bits (paranoid)
    3 No TLS certificate (not recommended)
    [1-3]: 2
(6) Add HTTP security headers?
    1 Strict    [enforce HTTPS] [disable: ext-resource, inline-css, iframes]
    2 Loose     [enforce HTTPS] [disable: ext-resource, iframes] [enable: inline-css]
    3 Poor      [enforce HTTPS] [enable: ext-resource, inline-css, iframes]
    4 Weak      [allow HTTP]    [enable: ext-resource, inline-css, iframes]
    5 None      [disable: HTTP security headers]
    [1-5]: 1
(7) Add logging?
    1 Error logging
    2 Access logging
    3 Error and access logging
    4 No logging
    [1-4]: 1
(8) Restart apache after creation? [yes/no]: yes
(9) Does the following configuration look reasonable?
    ############################################################################
    # ServerName:         nozel.org
    # ServerAlias         www.nozel.org
    # DocumentRoot:       /usr/local/www/nozel.org
    # TLS Certificate:    RSA 4096
    # Security headers:
    #   Strict-Transport-Security: max-age=31536000; includeSubDomains;
    #   X-Frame-Options "DENY"
    #   X-XSS-Protection: "1; mode=block"
    #   X-Content-Type-Options "nosniff"
    #   X-Permitted-Cross-Domain-Policies: none
    #   Referrer-Policy same-origin
    #   Content-Security-Policy: default-src https://nozel.org
    # Logging:            Error logging
    # Restart Apache:     Yes
    ############################################################################
    (yes/no): yes
Creating nozel.org.conf in /usr/local/etc/apache24/Includes
Setting ownership and permissions on nozel.org.conf
Adding VirtualHost for http requests to nozel.org.conf
Adding VirtualHost for https requests to nozel.org.conf
```

### Generated file
The example below uses the 'strict' HTTP Security Headers profile.
```
# apache configuration file generated by Sparry
<VirtualHost *:80>
    ServerName nozel.org
    ServerAlias www.nozel.org
    DocumentRoot /usr/local/www/nozel.org

    # Logging
    ErrorLog "/var/log/httpd-nozel.org-error.log"
    CustomLog "/var/log/httpd-nozel.org-access.log" common

    # HTTP Security Headers
    Header always set HTTP Security Headers
    Header always set Strict-Transport-Security: max-age=31536000; includeSubDomains;
    Header always set X-Frame-Options "DENY"
    Header always set X-XSS-Protection: "1; mode=block"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Permitted-Cross-Domain-Policies: none
    Header always set Referrer-Policy same-origin
    Header always set Content-Security-Policy: default-src https://nozel.org

    # Rewrite requests to HTTPS
    RewriteEngine on
    RewriteCond %{SERVER_NAME} =nozel.org [OR]
    RewriteCond %{SERVER_NAME} =nozel.org
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,QSA,R=permanent]
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName nozel.org
    ServerAlias www.nozel.org
    DocumentRoot /usr/local/www

    # Logging
    ErrorLog "/var/log/httpd-nozel.org-error.log"
    CustomLog "/var/log/httpd-nozel.org-access.log" common

    # HTTP Security Headers
    Header always set HTTP Security Headers
    Header always set Strict-Transport-Security: max-age=31536000; includeSubDomains;
    Header always set X-Frame-Options "DENY"
    Header always set X-XSS-Protection: "1; mode=block"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Permitted-Cross-Domain-Policies: none
    Header always set Referrer-Policy same-origin
    Header always set Content-Security-Policy: default-src https://nozel.org
</VirtualHost>
</IfModule>
```

## Support
If you have questions/suggestions about Sparry or find bugs, please let us know via the issue tracker.
