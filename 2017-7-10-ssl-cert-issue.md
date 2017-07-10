---
title: 2017-7-10 LE 证书签发
date:  2017-07-10 13:23:35
tags: Note
categories: Note
grammar_cjkRuby: true
---

### issue
````shell
acme.sh --issue -d www.example.com -d example.com -d www.v3.example.com \
-d api.example.com \
-d m.example.com -d m.v3.example.com \
-d admin.example.com -d admin.v3.example.com -d mail.example.com \
-d img.example.com -d static.example.com -d help.example.com \
-d git.shanjing-inc.com \
-d labs.shanjing-inc.com \
-d sentry.shanjing-inc.com \
--dns dns_dp
````

### 触发证书的安装命令

````shell
acme.sh --install-cert -d www.example.com \
--keypath  /var/build_and_deploy/config/sslcerts/example/example.com.key.pem  \
--fullchainpath /var/build_and_deploy/config/sslcerts/example/example.com.cert.pem \
--reloadcmd  "/root/.acme.sh/ssldeploy.sh"
````
/ssldeploy.sh内容：

````shell
#!/bin/bash
echo "aaa" >> /root/.acme.sh/fengittest.conf
docker run --rm -v /root/.acme.sh:/var/sslcert -v /var/build_and_deploy:/external_file -e "ANSIBLE_CONFIG=/external_file/ansible/common/ansible.cfg" daocloud.io/hmilyfyj/php-fpm ansible-playbook -i /external_file/ansible/common/allhosts /external_file/ansible/company/ssl-deploy/deploy.yml
````
