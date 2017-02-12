title: 服务端支持 IPV6
date: 2017-02-12 20:23:09
tags: [ipv6,iOS,nginx]
categories: ipv6
---

公司提交的 iOS 客户端需要ipv6的支持。遂记录下后端支持的过程。

<!-- more -->

---

## 服务端开通ipv6

### 方法1

有 ipv6 原生的国外vps，做 `nginx` 反代。

nginx 配置如下：

````nginx
server{
	listen    80;
	#listen   2607:8700:101:6522::1:80;
	listen 	  [::]:80;
	listen    443 ssl;
	listen 	  [::]:443 ssl;
	#listen   2607:8700:101:6522::1:443 ssl;
	server_name api.tbxzs.com m.v3.tbxzs.com ipv6.tbxzs.com static.tbxzs.com;
	ssl_certificate /usr/develop/nginx/sslkey/XX.crt;  #(证书公钥）
	ssl_certificate_key /usr/develop/nginx/sslkey/XX.key;  #(证书私钥）
	ssl_session_timeout 5m;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	#ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
	#ssl_prefer_server_ciphers on;
	#add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
	#add_header Content-Security-Policy upgrade-insecure-requests;
	if ( $scheme = http ) {
		#rewrite ^/(.*) https://$server_name/ permanent;
	}
	location / {
		proxy_pass http://xxxxIP:xx（一般是80）/;
		proxy_set_header HOST $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
````

### 方法2

