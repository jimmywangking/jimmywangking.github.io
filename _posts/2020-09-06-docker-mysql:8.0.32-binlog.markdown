---
layout: post
title:  "Docker build MySQL 8.0.21 with BinLog"
date:   2020-09-06 20:01:37 +0800
categories: 
---

1. 从docker hub 拉取mysql 8.0.21
docker pull msyql:8.0.21
也可以创建容器时候自动拉取

2. 创建config文件,用于配置mysql
建议放到个人目录,很多guide都是放到/home/data下,实际上macOs等会出现权限问题
命令行执行:
2.1 vi Users/wang/mysql8/config/my.cnf

2.2 把配置放入文件并保存
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Custom config should go here
!includedir /etc/mysql/conf.d/

#mysql8 authentication
default_authentication_plugin= mysql_native_password

#bin log setting
log-bin = /var/lib/mysql/mysql-bin
binlog-format = ROW
server-id = 1

3. 创建Mysql8容器
docker run \
    -p 3307:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v /Users/wang/mysql8/data:/var/lib/mysql:rw \
    -v /Users/wang/mysql8/log:/var/log/mysql:rw \
    -v /Users/wang/mysql8/config/my.cnf:/etc/mysql/my.cnf:rw \
    -v /etc/localtime:/etc/localtime:ro \
    --name mysql8 \
    --restart=always \
    -d mysql:8.0.21

4. 启动容器并进入Mysql8查看binlog是否开启成功
4.1 启动容器 docker start mysql8
4.2 进入容器 docker exec -ti mysql8 bash
4.3 登陆mysql  mysql -u root -p
4.4 查看binLog是否开启成功 show variables like ‘log_bin’


5. 网上有后期追加binlog的方式, 有一种通过容器内echo追加my.cnf里参数的方式貌似可行,大家可以自己尝试,成长在于折腾! 


-------------------------------------------------------------------------------:
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

