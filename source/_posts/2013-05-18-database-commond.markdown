---
layout: post
title: "database commond"
date: 2013-05-18 12:55
comments: true
categories: database
---
### PostgreSQL
create user
{% highlight bash %}
CREATE USER davide WITH PASSWORD 'jw8s0F4';
{% endhighlight %}

create database
{% highlight bash %}
CREATE DATABASE dbname;
CREATE DATABASE dbname OWNER username TABLESPACE spacename;
CREATE DATABASE dbname ENCODING 'LATIN1' TEMPLATE template0;
{% endhighlight %}

login
{% highlight bash %}

psql "dbname=databasename host=host user=user password=pwd port=5432 sslmode=require"
{% endhighlight %}

create table
{% highlight bash %}
create table testtbl(num int, name varchar(50));
{% endhighlight %}

show
{% highlight bash %}
\d;
\d testtbl;
{% endhighlight %}

各テーブルのレコード数:
{% highlight bash %}
SELECT COUNT(*) FROM テーブル名;
{% endhighlight %}

すべてのレコード数
{% highlight bash %}
SELECT T2.relname , T2.reltuples FROM pg_stat_user_tables AS T1 INNER JOIN pg_class AS T2 ON T1.relname = T2.relname ORDER BY T2.relname;
{% endhighlight %}

others
{% highlight bash %}
insert into testtbl values(1,'kaku');
select * from testtbl;
update testtbl set name='kaku' where num=1;
delete from testtbl where num=1;
drop table testtbl;
{% endhighlight %}

ログアウト
{% highlight bash %}
\q
{% endhighlight %}

### MySQL
{% highlight bash %}
mysql -h host -u user -p
grant all privileges on test.* to centos@localhost identified by 'centospass';  ←testデータベースへの全てのアクセス権限を持った、新規ユーザcentosを登録
select user from mysql.user where user='centos';　←centosユーザ登録確認
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');パースワードを変える
create database name;
drop database name; 削除する
show databases;
use database name;
show tables;
select * from table name;
drop table name;
create table events (id INTEGER NOT NULL AUTO_INCREMENT, date VARCHAR(20), event_type VARCHAR(20), count_type VARCHAR(20), value INT, primary key (id));
create table errors (id INTEGER NOT NULL AUTO_INCREMENT, date VARCHAR(20), total INT, primary key (id));
{% endhighlight %}

mysqlのsocket問題
{% highlight bash %}
ln -s /var/run/mysqld/mysqld.sock /tmp/mysql.sock
{% endhighlight %}

config
{% highlight bash %}
yum -y install mysql-server
/etc/my.cnf　←　MySQL設定ファイル編集
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Default to using old password format for compatibility with mysql 3.x
# clients (those using the mysqlclient10 compatibility package).
old_passwords=1
default-character-set = utf8　←　追加(MySQLサーバーの文字コードをUTF-8にする)

以下を追加(MySQLクライアントの文字コードをUTF-8にする)
[mysql]
default-character-set = utf8
[root@centos ~]# /etc/rc.d/init.d/mysqld start　←　MySQL起動
[root@centos ~]# chkconfig mysqld on　←　MySQL自動起動設定
[root@centos ~]# mysql_secure_installation　←　MySQL初期設定
{% endhighlight %}

### dynamodb
When your read or write throughput exceed your table's allowed provisioning, DynamoDB will wait on connections until throughput is available again. This will appear as very, very slow requests and can be somewhat frustrating. Partitioning significantly increases the amount of throughput tables will experience; though DynamoDB will ignore keys that don't exist, if you have 20 partitioned keys representing one object, all will be retrieved every time the object is requested. Ensure that your tables are set up for this kind of throughput, or turn provisioning off, to make sure that DynamoDB doesn't throttle your requests.

DynamoDB通过random pattern来进行读写。在内部，hash keys在各个server里面是分散的，从两个连续的服务器读取要不从一个服务器读取两次要快。当然，我们有许多程序请求一个key比请求其他的key要频繁。在DynamoDB里面这样做会导致底吞吐量和慢的反应。我们应该避免。
dynamoid试图通过使用一个分区策略来避免这个问题，分区策略通过DynamoDB的服务器来随机的分割key。每个id在保存的时候会分配到一个附加号码（默认是0-199，但是可以增加partition size来增加这个号码）。当read的时候，200个hash同时被取回来，最近的更新的那个会被传给程序。
当你读或者写的吞吐量超过你的table允许的provision的时候，dynamodb会等待连接，直到吞吐量再次可用。这样的话请求就会变得很慢，并且会让人很沮丧。分割意味着增加了吞吐量。尽管dnnamodb会忽略不存在的key，如果你有20个被分割的key来表示1个object，那么这个object每次被请求的时候都会有20个请求。一定要确保dynamodb不会节流你的请求。
