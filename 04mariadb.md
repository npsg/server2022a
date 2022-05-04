# MariaDBのインストール
## mariadbのインストール
```
> yum install mariadb-server
```
## mariadbの起動
```
> systemctl start mariadb
> systemctl enable mariadb
> systemctl is-enabled mariadb
```
## /etc/my.cnf の編集
``` Apache 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
character-set-server=utf8
```
```
systemctl restart mariadb
```
## rootパスワードの変更
mysqlクライアントで変更
```
MariaDB [(none)]> set password = password('tebasaki');
```
パスワードありでログインする
```
mysql -u root -p
````
