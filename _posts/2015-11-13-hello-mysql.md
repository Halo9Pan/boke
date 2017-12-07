---
layout:     post
title:      MySQL 简介
date:       2015-11-06
categories: Development
---

### 安装

### 初始化
#### my.cnf/my.ini
{% highlight ini %}
[client]
user                    = root
password                = XXXXXXXX
port                    = 11806
socket                  = F:/Eris/mysql/socket
default-character-set   = utf8

# The MySQL server
[mysqld]
basedir                 = F:/Luna/mysql/
datadir                 = F:/Eris/mysql/data
port                    = 11806
socket                  = F:/Eris/mariadb/socket
key_buffer_size         = 16K
max_allowed_packet      = 1M
table_open_cache        = 4
sort_buffer_size        = 64K
read_buffer_size        = 256K
read_rnd_buffer_size    = 256K
net_buffer_length       = 2K
thread_stack            = 128K
sql_mode                = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
skip-external-locking
character-set-server    = utf8
collation-server        = utf8_unicode_ci
lower_case_table_names  = 1

# Don't listen on a TCP/IP port at all. This can be a security enhancement,
# if all processes that need to connect to mysqld run on the same host.
# All interaction with mysqld must be made via Unix sockets or named pipes.
# Note that using this option without enabling named pipes on Windows
# (using the "enable-named-pipe" option) will render mysqld useless!
# 
#skip-networking
server-id	              = 1

# Uncomment the following if you want to log updates
#log-bin=mysql-bin

# binary logging format - mixed recommended
#binlog_format=mixed

# Causes updates to non-transactional engines using statement format to be
# written directly to binary log. Before using this option make sure that
# there are no dependencies between transactional and non-transactional
# tables such as in the statement INSERT INTO t_myisam SELECT * FROM
# t_innodb; otherwise, slaves may diverge from the master.
#binlog_direct_non_transactional_updates=TRUE

# Uncomment the following if you are using InnoDB tables
#innodb_data_home_dir               = F:/Eris/mysql/data
#innodb_data_file_path              = ibdata1:10M:autoextend
#innodb_log_group_home_dir          = F:/Eris/mysql/data
# You can set .._buffer_pool_size up to 50 - 80 %
# of RAM but beware of setting memory usage too high
innodb_buffer_pool_size             = 16M
# innodb_additional_mem_pool_size   = 2M
# Set .._log_file_size to 25 % of buffer pool size
innodb_log_file_size                = 5M
innodb_log_buffer_size              = 8M
#innodb_flush_log_at_trx_commit     = 1
#innodb_lock_wait_timeout           = 50

[mysqldump]
quick
max_allowed_packet      = 16M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates
default-character-set   = utf8
{% endhighlight %}

#### 初始化root密码
{% highlight sh %}
mysqladmin -u root -p flush-privileges password "new_pwd"
mysqladmin -u root -p flush-privileges password
{% endhighlight %}

#### 执行SQL
{% highlight sh %}
mysql -u root -p -e ""
{% endhighlight %}

#### 更新root密码
{% highlight sql %}
SELECT User,Host FROM mysql.user;

SET PASSWORD FOR 'root'@'127.0.0.1' PASSWORD('new_pwd');
SET PASSWORD FOR 'root'@'localhost' PASSWORD('new_pwd');
{% endhighlight %}

#### root本地登录、移除匿名用户
{% highlight sql %}
SELECT User,Host FROM mysql.user;

DROP USER 'root'@'%';
DROP USER ''@'localhost';
{% endhighlight %}

#### 创建用户
{% highlight sql %}
GRANT USAGE ON *.*
TO 'hello'@'localhost'
IDENTIFIED BY 'pass9USE';

GRANT SELECT ON *.* TO 'hello'@'localhost';

SHOW GRANTS FOR 'hello'@'localhost' \g

GRANT ALL ON *.* TO 'hello'@'localhost';
{% endhighlight %}


### 常用命令
{% highlight sql %}
SHOW DATABASES;
{% endhighlight %}

{% highlight sql %}
CREATE TABLE test.hello (hello_id INT, word TEXT);

SHOW TABLES FROM test;

USE test;
SHOW TABLES;
DESCRIBE hello;
{% endhighlight %}

{% highlight sql %}
INSERT INTO hello VALUES(100, 'World');
INSERT INTO hello VALUES(101, 'Welt');
INSERT INTO hello VALUES(102, 'Monde');
--INSERT INTO hello VALUES(101, '世界');
{% endhighlight %}

{% highlight sql %}
SOURCE ???.sql
{% endhighlight %}

#### 数据库创建
{% highlight sql %}
CREATE DATABASE hello;

DROP DATABASE hello;
CREATE DATABASE hello
CHARACTER SET utf8
COLLATE utf8_unicode_ci;
{% endhighlight %}

#### 创建表
{% highlight sql %}
CREATE TABLE guest (
id INT AUTO_INCREMENT PRIMARY KEY,
nick_name CHAR(64) UNIQUE,
visit_time INT);

DESCRIBE guest;

SHOW CREATE TABLE guest \g
{% endhighlight %}

{% highlight sql %}
CREATE TABLE vocabulary (
id BIGINT DEFAULT 0,
word CHAR(16) UNIQUE,
emotion_joy SMALLINT DEFAULT 0,
emotion_sad SMALLINT DEFAULT 0,
emotion_anger SMALLINT DEFAULT 0,
emotion_fear SMALLINT DEFAULT 0,
emotion_trust SMALLINT DEFAULT 0,
emotion_suspect SMALLINT DEFAULT 0,
emotion_expect SMALLINT DEFAULT 0,
emotion_surprised SMALLINT DEFAULT 0,
create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
update_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);

DESCRIBE vocabulary;

SHOW CREATE TABLE vocabulary \g
{% endhighlight %}

{% highlight sql %}
CREATE TABLE guest_isam (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nick_name CHAR(64) UNIQUE,
  visit_time INT NOT NULL DEFAULT 0
) ENGINE=MyISAM AUTO_INCREMENT=1000 DEFAULT CHARSET=gbk COLLATE=gbk_chinese_ci;
{% endhighlight %}

{% highlight sql %}
SHOW FULL COLUMNS
FROM guest LIKE 'nick' \g
{% endhighlight %}

#### 备份数据
[mysqldump](http://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)
{% highlight sh %}
mysqldump --user='hello' -p hello guest > hello-guest.sql
mysqldump --user='hello' -p hello > hello.sql
{% endhighlight %}
{% highlight sh %}
mysqldump --single-transaction --user='hello' -p hello > hello.sql
{% endhighlight %}

#### 还原数据
{% highlight sh %}
mysql --user='hello' -p hello guest < hello-guest.sql
mysql --user='hello' -p hello < hello.sql
{% endhighlight %}

#### 修改表
{% highlight sql %}
ALTER TABLE guest
ADD COLUMN description TEXT;
DESCRIBE guest;

ALTER TABLE vocabulary
ADD COLUMN description VARCHAR(255);
DESCRIBE vocabulary;
{% endhighlight %}

{% highlight sql %}
CREATE TABLE guest_like LIKE guest;
DESCRIBE guest_like;

INSERT INTO guest_like
SELECT * FROM guest;
SELECT * FROM guest_like;
DROP TABLE guest_like;

CREATE TABLE guest_create
SELECT * FROM guest;
DESCRIBE guest_create;
SELECT * FROM guest_create;
DROP TABLE guest_create;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
DROP COLUMN description;
DESCRIBE guest;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
ADD COLUMN nick_type CHAR(2) AFTER nick_name;
DESCRIBE guest;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
CHANGE COLUMN nick_type nick_type CHAR(4);
DESCRIBE guest;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
MODIFY COLUMN nick_type
ENUM('Auto','Manual')
AFTER nick_name;
DESCRIBE guest;
{% endhighlight %}

{% highlight sql %}
UPDATE guest SET visit_time=0 WHERE visit_time IS NULL;

ALTER TABLE guest MODIFY COLUMN visit_time INT NOT NULL DEFAULT 0;
DESCRIBE guest;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
AUTO_INCREMENT = 100;
INSERT INTO guest (nick_name, visit_time)
VALUES ('Garnett', 1),
('Messi', 3);
INSERT INTO guest (nick_name)
VALUES ('Ronaldo ');
SELECT * FROM guest;
{% endhighlight %}

### 数据操作

#### 插入数据
{% highlight sql %}
INSERT INTO guest (nick_name)
VALUES ('Ada'),
('David'),
('Robbie');

INSERT INTO guest (nick_name, visit_time)
VALUES ('Johnson', 1),
('John', 0),
('James', 0);

SELECT * FROM guest;
{% endhighlight %}

{% highlight sql %}
CREATE TABLE guest_new LIKE guest;
DESCRIBE guest_new;

ALTER TABLE guest_new
ADD COLUMN nick_name_new VARCHAR(127);

INSERT IGNORE INTO guest_new
(nick_name, nick_name_new, visit_time)
SELECT nick_name, nick_name, visit_time FROM guest;

SELECT * FROM guest_new;
{% endhighlight %}

{% highlight sql %}
REPLACE INTO guest (nick_name)
VALUES ('Ada'),
('David'),
('Robbie');

REPLACE INTO guest (nick_name, visit_time)
VALUES ('Johnson', 101),
('John', 100),
('James', 100);

REPLACE INTO guest (nick_name, nick_type)
VALUES ('Ada', 1),
('David', 1),
('Robbie',1 );

SELECT * FROM guest;
{% endhighlight %}

{% highlight sql %}
INSERT LOW_PRIORITY INTO guest (nick_name)
VALUES ('Aaric'),
('Aart'),
('Abbeygail');

-- INSERT DELAYED INTO guest (nick_name)
-- VALUES ('Abbot'), ('Abbott'), ('Abeodan');

INSERT INTO guest (nick_name) VALUES ('Aaric')
  ON DUPLICATE KEY UPDATE nick_name = CONCAT('!', nick_name);
SELECT * FROM guest;

DELETE FROM guest WHERE nick_name LIKE '!%';
SELECT * FROM guest;
{% endhighlight %}

{% highlight sql %}
ALTER TABLE guest
ADD COLUMN nick_name_used SMALLINT NOT NULL DEFAULT 1;
DESCRIBE guest;

INSERT INTO guest (nick_name) VALUES ('Aaric')
  ON DUPLICATE KEY UPDATE nick_name_used = nick_name_used + 1;

SELECT * FROM guest;
{% endhighlight %}

#### 查询数据
{% highlight sql %}
SELECT * FROM guest;

SELECT * FROM guest WHERE visit_time=1;
SELECT * FROM guest WHERE visit_time IN (1, 100);
SELECT * FROM guest WHERE nick_name LIKE 'A%';
SELECT * FROM guest WHERE nick_name REGEXP 'i|o';
SELECT * FROM guest WHERE nick_name NOT REGEXP 'i|o';
SELECT * FROM guest WHERE visit_time=1 AND nick_name LIKE 'A%';
SELECT * FROM guest WHERE visit_time=1 OR nick_name LIKE 'A%';

SELECT * FROM guest ORDER BY visit_time;

SELECT * FROM guest WHERE visit_time>=1 ORDER BY visit_time LIMIT 5;
SELECT * FROM guest WHERE visit_time>=1 ORDER BY visit_time DESC LIMIT 2;

SELECT COUNT(*) FROM guest;
SELECT * FROM guest GROUP BY visit_time;

SELECT nick_name, visit_time FROM guest;
SELECT nick_name AS 'Nick Name', visit_time AS 'Visit Times' FROM guest;

SELECT visit_time AS 'Visit Times',
COUNT(*) AS 'Number'
FROM guest
GROUP BY visit_time;
{% endhighlight %}

#### 更新数据
{% highlight sql %}
SELECT * FROM guest;
UPDATE guest
SET visit_time = 10
WHERE nick_name_used > 1;
SELECT * FROM guest;
{% endhighlight %}

#### 删除数据
{% highlight sql %}
SELECT * FROM guest;
DELETE FROM guest
WHERE visit_time = 0
AND nick_name_used = 1;
SELECT * FROM guest;
{% endhighlight %}

#### 联合查询
{% highlight sql %}
{% endhighlight %}

