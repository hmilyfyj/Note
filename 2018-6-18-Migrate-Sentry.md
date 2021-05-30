---
title: Sentry 服务器迁移
date: 2018-06-18 10:36:58
tags: Sentry
categories: Sentry
---

Sentry 在高并发期还是很占资源的。所以将其迁移出业务服务器。

<!-- more -->

---

### 清理数据

```
sentry cleanup -days 30 -q
vacuumdb -U postgres -d 0532Shanjing -t nodestore_node -v -f --analyze
```

### 备份数据
```
//方法 1：数据库全量导出
pg_dump  -U postgres 0532Shanjing -f /backup/postgres_0532Shanjing_2018-06-16_1.dump

//方法 2：使用官方的导出功能，仅保留关键配置信息。
sentry --config /etc/sentry export  --exclude
```

将导出的文件文件传输到新的机器上。

### 恢复

#### 复制数据

```
scp /data/postgres/postgres_0532Shanjing_2018-06-16.dump  root@10.116.204.151:/data/postgres/
```

# 安装教程
https://g.yuque.com/robinson/fe-pro/per2dt

# 修改 build 过程中的源
RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

# 更新 docker-compose 的方法
curl -L https://get.daocloud.io/docker/compose/releases/download/1.28.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 加速 docker 的各种方法
https://yeasy.gitbook.io/docker_practice/install/mirror
# 大工程
8.22 =》 21

相关资料：
https://forum.sentry.io/t/migration-from-8-xx-to-20-xx/11795
https://forum.sentry.io/t/moving-to-a-new-server-from-9-0-0-to-9-1-2-docker/10640
https://github.com/getsentry/onpremise/issues/357
https://github.com/getsentry/onpremise/issues/387
https://www.kkzxak47.com/
https://medium.com/@avigny/sentry-on-premise-migration-dc0e42f85af4
https://blog.csdn.net/leirace/article/details/90975960
https://sucipto.id/2018/07/26/upgrade-sentry-on-premise-from-8-22-to-9-0/
https://github.com/getsentry/onpremise/issues/128
https://www.codeleading.com/article/92151071933/
https://forum.sentry.io/t/how-to-upgrade-from-sentry-9-to-10/8560


#### 导入数据
```
docker run -it --rm -e SENTRY_SECRET_KEY='' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade
```

# 启动过程中占用了较多资源
利用 docker-compose 限制资源的方法：https://www.cnblogs.com/peitianwang/p/12173296.html
# onpremise 相关资料
菜鸟：https://www.runoob.com/postgresql/postgresql-tutorial.html
进入本地数据库：psql -U postgres

# PostgreSQL
DETAIL:  The data directory was initialized by PostgreSQL version 9.5, which is not compatible with this version 9.6.20.
https://github.com/getsentry/onpremise/issues/368
https://github.com/docker-library/postgres/issues/682
https://www.codenong.com/17822974/

官方脚本已支持自动升级，不用自己处理。

## 开启日志
查看数据
https://www.cnblogs.com/qianxunman/p/12149586.html

```
log_statement = 'all'

# This is used when logging to stderr:
logging_collector = on        # Enable capturing of stderr and csvlog
                    # into log files. Required to be on for
                    # csvlogs.
                    # (change requires restart)

# These are only used if logging_collector is on:
log_directory = 'log'            # directory where log files are written,
                    # can be absolute or relative to PGDATA
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'    # log file name pattern,
                    # can include strftime() escapes
log_file_mode = 0600            # creation mode for log files,
                    # begin with 0 to use octal notation
#log_truncate_on_rotation = off        # If on, an existing log file with the
                    # same name as the new log file will be
                    # truncated rather than appended to.
                    # But such truncation only occurs on
                    # time-driven rotation, not on restarts
                    # or size-driven rotation.  Default is
                    # off, meaning append to existing files
                    # in all cases.
#log_rotation_age = 1d            # Automatic rotation of logfiles will
                    # happen after that time.  0 disables.
#log_rotation_size = 10MB        # Automatic rotation of logfiles will
                    # happen after that much log output.
                    # 0 disables.
```
存储路径：/var/lib/postgresql/data/log/postgresql-2021-02-02_000000.log

cat postgresql-2021-02-04_000000.log | grep 'ERROR\|DETAIL\|STATEMENT'


前台创建项目时，收到系统报错：A project with this slug already exists. 错误过于简洁，于是查询 postgres 的全量日志。发现报错内容是：
```
ERROR:  duplicate key value violates unique constraint "sentry_project_pkey"
DETAIL:  Key (id)=(14) already exists
```

所以只要把自增 id 设置为 max(id) 就可以了。

```
SELECT setval(pg_get_serial_sequence('sentry_project', 'id'), (SELECT MAX(id) FROM sentry_project));
```

sentry_onpremise_postgres_1
psql -U postgres
\c postgres
SELECT setval(pg_get_serial_sequence('auth_permission', 'id'), (SELECT MAX(id) FROM auth_permission));

```
INSERT INTO "sentry_projectkey" ("project_id", "label", "public_key", "secret_key", "roles", "status", "date_added", "rate_limit_count", "rate_limit_window", "data") VALUES (34, 'Default', 'fdc72b02b13540d18839385326818814', '8b434e1487d642e080c8bc864e6c0897', 1, 0, '2021-02-03T12:41:20.296606+00:00'::timestamptz, NULL, NULL, '{}') RETURNING "sentry_projectkey"."id"

duplicate key value violates unique constraint "sentry_projectkey_pkey"

SELECT setval(pg_get_serial_sequence('sentry_projectkey', 'id'), (SELECT MAX(id) FROM sentry_projectkey));

SELECT setval(pg_get_serial_sequence('sentry_team', 'id'), (SELECT MAX(id) FROM sentry_projectkey));

SELECT setval(pg_get_serial_sequence('sentry_organizationmember_teams', 'id'), (SELECT MAX(id) FROM sentry_organizationmember_teams));

SELECT setval(pg_get_serial_sequence('sentry_projectteam', 'id'), (SELECT MAX(id) FROM sentry_projectteam));
SELECT setval(pg_get_serial_sequence('sentry_organizationoptions', 'id'), (SELECT MAX(id) FROM sentry_organizationoptions));


SELECT setval(pg_get_serial_sequence('sentry_rule', 'id'), (SELECT MAX(id) FROM sentry_rule));
```

设置自增 id 的相关教程：
https://stackoverflow.com/questions/19135161/django-db-utils-integrityerror-duplicate-key-value-violates-unique-constraint
https://www.imooc.com/wenda/detail/565590
https://cloud.tencent.com/developer/ask/27217
https://gist.github.com/henriquemenezes/962829815e8e7875f5f4376133b2f209
https://blog.csdn.net/baidu_18607183/article/details/51181317
https://blog.csdn.net/chen_2890/article/details/94727594?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control
https://www.jb51.net/article/203416.htm
https://www.runoob.com/postgresql/postgresql-constraints.html

\l 库
\c 进入
\d 表 查看表信息

```
django.db.utils.IntegrityError: UniqueViolation('duplicate key value violates unique constraint "django_
content_type_app_label_model_76bd3d3b_uniq"\nDETAIL:  Key (app_label, model)=(sentry, groupresolution) a
lready exists.\n',)
SQL: UPDATE "django_content_type" SET "app_label" = %s, "model" = %s WHERE "django_content_type"."id" = 
%s
```

https://www.kkzxak47.com/2020/08/29/sentry-onpremise%E8%BF%81%E7%A7%BB%E8%B8%A9%E5%9D%91/

TRUNCATE TABLE django_content_type；
TRUNCATE TABLE django_content_type CASCADE;
TRUNCATE TABLE sentry_projectoptions CASCADE;
# 请空数据
docker-compose down && docker volume prune && rm -rf data/postgres
mkdir -p data/{sentry,postgres}


# 
docker-compose run --rm web config generate-secret-key

docker-compose run --rm web upgrade

docker-compose up -d

cd /var/lib/sentry/files
sentry import sentry_u.json


sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication` > sentry_u.json

sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication > sentry_u_new_8.22.json

sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication > sentry_u_new_9.json

sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication > sentry_u_new_9.json

sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication > sentry_u_new_9.1.2.json
sentry --config /etc/sentry export  > sentry_full_new_9.1.2.json


sentry --config /etc/sentry export  --exclude savedsearch,rule,permission,migrationhistory,contenttype,option,site,userip,useroption,projectoption,counter,organizationmemberteam,organizationmember,team,organizationoption,projectteam,authenticator,organization,projectbookmark,apiapplication > sentry_u_new_9.1.2.json
sentry --config /etc/sentry export  > sentry_full_new_20.json


# 版本问题
# 换源
RUN cat /etc/apt/sources.list


老板，我建议给微红娘的服务器升级 cpu，因为微红娘的 cpu 无法满足 sentry 的运行，官方推荐是 4c8g，我目前通过 docker 限制了 sentry 的占用，避免对服务器产生影响，限制后的 sentry 运行非常慢，启动就要二三十分钟。

sentry 配置迁移好了，从 8.22 升到了 21.0，测试地址：

# github 加速
https://segmentfault.com/a/1190000023411392
https://hub.fastgit.org/
https://github.com.cnpmjs.org/
https://github.zhlh6.cn/
https://github.com/chenxuhua/issues-blog/issues/3

# 解决

error: could not parse json config file (file /work/.relay/credentials.json)

经排查发现是 json 配置文件掺入了其他内容，清理掉即可。目录，宿主机的 ./relay 文件夹。

# 
```
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```

aliyun cloud linux 无法直接安装 docker，并且无法安装最新版 docker，随指定旧版 docker：
yum install docker-ce-19.03.13-3.el7 docker-ce-cli-19.03.13-3.el7 containerd.io


CentOS7使用yum方式安装指定版本的Dockerhttps://www.yht7.com/news/16573

在linux下如何使用yum查看安装了哪些软件包
 https://blog.csdn.net/wenwenxiong/article/details/51785221

官方安装 docker-io
https://tech.antfin.com/docs/2/51853

菜鸟教程
https://www.runoob.com/docker/centos-docker-install.html

同样遇到问题的反馈：
https://segmentfault.com/q/1010000019893786
# sentry wxwork
https://www.jianshu.com/p/a09aa15ecd6f
http://www.iocoder.cn/Sentry/install/?vip
https://work.weixin.qq.com/help?person_id=0&doc_id=423&helpType=exmail

# mail.backend: 'smtp'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.exmail.qq.com'
mail.port: 587(端口一定要用587， sentry只支持tls而非ssh)
mail.username: 'example@example.com'
mail.password: 'password'
mail.use-tls: true
# The email address to send on behalf of
mail.from: 'example@example.com'
