title: centos 下安装 pyspider
date: 2015-12-18 20:21
tags: [python,爬虫,折腾]
categories: python
---
### 安装PIP

	wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate

	python get-pip.py


### 安装依赖：

    yum install python-devel zlib-devel libxslt-devel  libxml2-dev
    
    sudo pip install -r requirements.txt
    
    pip install mysql-connector-python==2.0.4 --allow-external mysql-connector-python
    
    Error: pg_config executable not found.
    
    pip install mysql-connector-python==2.1.2 --allow-external mysql-connector-python
    
    yum install postgresql-devel

