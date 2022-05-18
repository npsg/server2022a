# PHP・Laravelインストール
## Remi リポジトリを利用するためには、先に EPEL リポジトリを使えるようにする
```
yum install epel-release
```
## 次に Remi リポジトリを取得
```
yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```
## PHP のバージョン 7.4をインストール
```
yum install php74
```
## PHP のコマンドラインプログラムを確認し、phpのバージョンを確認
- /etc/profile.d/に移動
- php74_env.shを作成し、以下を入れる
```
export PATH=$PATH:/opt/remi/php74/root/usr/bin
```
- sourceコマンドで実行する
> sourceコマンドでは現在のシェルで実行するため、シェル環境の影響を受けます。
- $PATHを確認する
```
> echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/opt/remi/php74/root/usr/bin
```
- phpコマンドでバージョンを確認する
```
> php -v
PHP 7.4.29 (cli) (built: Apr 12 2022 10:55:38) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```
## php.iniの設定
- /etc/opt/remi/php74 のphp.iniを編集
```Apache
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = Asia/Tokyo
[mbstring]
; language for internal character representation.
; This affects mb_send_mail() and mbstring.detect_order.
; http://php.net/mbstring.language
mbstring.language = Japanese
```
## PHP Extensionsインストール
```
> yum -y install php74-php php74-php-bcmath php74-php-mbstring php74-php-mcrypt php74-php-xml php74-php-mysqlnd php74-php-gd php74-php-zip php74-php-process php74-php-tidy php74-php-tokenizer php74-php-opcache php74-php-intl php74-php-xdebug
```
## Composer のインストール
```
> curl https://getcomposer.org/installer | php
```
composer.pharをコマンドで利用できるように移動
```
> mv composer.phar /usr/local/bin/composer
```
## Laravel プロジェクトの生成
```
> cd /var/www
> composer create-project laravel/laravel sample --prefer-dist "7.*"
```
## Apache から Laravel プロジェクトを呼び出す設定
1. ドキュメントルートを/var/www/sample/public にする
```
vi /etc/httpd/conf/httpd.conf
```
```Apache
DocumentRoot "/var/www/sample/public"
```
2. /var/www/sample/public へのアクセスを許可する
3. .htaccess による上書き設定(AllowOverride)を All にする
``` Apache
<Directory "/var/www/sample/public" >
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    #Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride All

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>

```
4. プロジェク トのキャッシュディレクトリ(bootstrap/cache)とプロジェクトのログディレクトリ (storage)以下のディレクトリに書き込み権限を設定
```
chmod -R 777 /var/www/sample/bootstrap/cache
chmod -R 777 /var/www/sample/storage
```
5. ブラウザからアクセスする

## 補足：laravelを使っている場合のIP制限（例．192.168.64.3はCentOSのIP）
- /etc/httpd/conf/httpd.confを編集
  - Require が重複しないように注意
```
<Directory "/var/www/sample/public" >
    AllowOverride All
    Require ip 192.168.64.3
</Directory>
```
- 403だとテストページが開いてしまうので、/etc/httpd/conf.d/welcome.confを編集し、コメントにする
```
<LocationMatch "^/+$">
#    Options -Indexes
#    ErrorDocument 403 /.noindex.html
</LocationMatch>
```
