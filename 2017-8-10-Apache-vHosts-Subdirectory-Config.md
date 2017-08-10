---
title: 2017-8-10-Apache-vHosts-Subdirectory-Config
date:  2017-08-10 10:43:12
tags: Note
categories: Note
grammar_cjkRuby: true
---



<!-- more -->

---

``` apache
<VirtualHost *:80>
    ServerAdmin webmaster@lara.loc
    DocumentRoot "E:\xampp\htdocs\old-proj\api"
    ServerName api.example.com
    ServerAlias api.example.com
    SSLEngine on
    SSLCertificateFile "conf/ssl.crt/server.crt"
    SSLCertificateKeyFile "conf/ssl.key/server.key"

    Alias /v3 E:\xampp\htdocs\old-proj\api_v3
    #AliasMatch ^.*$ "E:\xampp\htdocs\old-proj\api_v3\index.php"
    <Directory "E:\xampp\htdocs\old-proj\api_v3">
            DirectoryIndex index.php
            AllowOverride All
            #Order allow,deny
            Allow from all
    </Directory>
</VirtualHost>
```
