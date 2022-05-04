# ApacheHTTPサーバー構築
## インストール前の確認

```sh
rpm -q httpd
パッケージ httpd はインストールされていません。
```
## パッケージからのインストール
```
> yum -y install httpd
```
> systemctlの使い方は、DNSサーバーの時と同じ

## firewall許可
```
> firewall-cmd --zone=public --add-service=http --permanent
> firewall-cmd --zone=public --add-service=https --permanent
> firewall-cmd --reload
> firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s4
  sources: 
  services: dhcpv6-client dns http https ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

## SELinuxをpermissiveモードに変更
/etc/sysconfig/selinux を編集  
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
この後、「shutdown -r now」して、再起動して、もう１度「firewall-cmd --list-all --zone=public」して、設定が残っているかどうか確認

## 起動
```
> systemctl start httpd
> systemctl status httpd
> systemctl enable httpd 
> systemctl is-enabled httpd 
```
## /etc/httpd/conf/httpd.confを編集
### 基本設定
- 行単位：　ディレクティブ 設定値
- ブロック単位  
 <ディレクティブ 対象>  
 </ディレクティブ>
### コメント
 #以降をコメントとして扱う

## 代表的なディレクティブ
- ServerRoot ディレクティブ：各種ディレクトリの基点となるディレクトリ  
ServerRoot "/etc/httpd"
- Listen ディレクティブ：接続を待ち受けるインターフェースの IP アドレスとポート番号を指定  
Listen 80
- User ディレクティブ/Group ディレクティブ：httpd を実行するときの実行ユーザ/グループを指定  
User apache  
Group apache  
- DocumentRoot ディレクティブ：公開するコンテンツを置くディレクトリのルート  
DocumentRoot "/var/www/html"
- DirectoryIndex ディレクティブ：ディレクトリ名までしか指定されていない場合に表示するファイル  
`<IfModule dir_module>`  
    DirectoryIndex index.html  
`</IfModule>`
- Directory ディレクティブ：ブロック単位で指定するディレクティブ  
範囲が限定される方の指定が上書きされて優先  
`<Directory "/var/www/html">`  
Options Indexes FollowSymLinks  
AllowOverride None  
Require all granted  
`</Directory>`
- Options ディレクティブ
  - FollowSymLinks：ディレクトリ下にシンボリックリンクがある場合にそれを辿る
  - Indexes：URL で指定したファイルパスがディレクトリ名までしか指定がな く、かつ DirectoryIndex で指定されたファイルが無い場合に、 そのディレクトリの一覧を表示する
- AllowOverride None：.htaccess では何も設定できません。
- Require all granted：ドキュメントルート以下のすべてのディレクトリに対してすべてのアクセスを許可
  - Require ip 192.168.56.などと設定すると、IP アドレスが 192.168.56.0~192.168.56.255 の範囲にあるホストからは WWW サー バーへのアクセスが許可されるが、それ以外のホストから接続されると 403 Forbidden の エラーが返される。

## SSLの設定
### mod_sslのインストール
インストールが終わると、/etc/httpd/conf.d/ssl.confが配置される。
```
> yum install mod_ssl
```
### /etc/httpd/conf.d/ssl.conf ファイルを確認
SSLEngine on  
SSLCertificateFile /etc/pki/tls/certs/localhost.crt  
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key  

## 証明書要求の作成
### /etc/pki/tls/openssl.cnf ファイルを編集
- コメントを外す  
req_extensions = v3_req # The extensions to add to a certificate request
- v3_req セクションに subjectAltName = @alt_names を追加し、alt_names セクションを作成し、証明書で承認されたいドメイン名を定義  

```Apache
[ v3_req ]

# Extensions to add to a certificate request

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment

subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.example.jp
```
### /etc/pki/tls/miscに移動し、「./CA -newreq」とコマンド 入力
```
[root@localhost misc]# ./CA -newreq
Generating a 2048 bit RSA private key
..................+++
..+++
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:Tokyo
Locality Name (eg, city) [Default City]:Minato
Organization Name (eg, company) [Default Company Ltd]:example
Organizational Unit Name (eg, section) []:example
Common Name (eg, your name or your server's hostname) []:www.example.jp
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```
### 生成できているか確認する
```
> openssl req -in newreq.pem -text
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=JP, ST=Tokyo, L=Minato, O=example, OU=example, CN=www.example.jp
        …略
```

### /etc/httpd/conf.d/ssl.conf ファイルを編集
SSLEngine on  
SSLCertificateFile /etc/pki/tls/misc/newcert.pem
SSLCertificateKeyFile /etc/pki/tls/misc/newkey.pem  

### 秘密鍵からパスフレーズを抜き、Apacheの再起動などのときにパスワードを入力しなくてもよくなる
openssl rsa -in newkey.pem -out newkey_nopass.pem
- CA の証明書をインポートするときに、拡張子を crt にしておくとダブルクリックするだけ でインポート処理できる

> 証明書要求と認証局の関係  
> https://www.infraexpert.com/study/security6.html

## プライベート認証局を利用
### /etc/pki/tls/openssl.cnf の設定
```Apache
[ usr_cert ]


# These extensions are added when 'ca' signs a request.

# This goes against PKIX guidelines but some CAs do it and some software
# requires this to avoid interpreting an end user certificate as a CA.

basicConstraints=CA:TRUE

[ CA_default ]
# Extension copying option: use with caution.
copy_extensions = copy

```
### 認証局の証明書と秘密鍵の生成
その後/etc/pki/tls/miscに移動し、「./CA -newca」とコマンド入力。認証局の証明書が/etc/pki/CA/cacert.pem というファイル名で生成され、認証局の秘密鍵が /etc/pki/CA/private/cakey.pem というファイル名で生成

### 認証局として証明書要求(CSR)に署名
/etc/pki/tls/miscに移動し、「./CA -sign」とコマンド入力。認証局の秘密鍵のパスフレーズが要求されて、後は YES と答えていくだけで証明書 (newcert.pem)が生成

### apache再起動する
```
> systemctl restart httpd
Enter SSL pass phrase for localhost.localdomain:443 (RSA) : ********
> systemctl status httpd
```

https://192.168.64.3でアクセスできる

## 特定のパスへのアクセスに HTTPS 接続を強制
/etc/httpd/conf.d/rewite.conf を編集
```apache
RewriteEngine On
RewriteCond %{SERVER_PORT} !^443$
RewriteRule ^/login/(.*)?$ https://%{HTTP_HOST}/login/$1 [R,L]
```
## ログファイルの確認
/etc/httpd/conf/httpd.confをviで開き、以下を確認
```Apache
CustomLog "logs/access_log" combined
ErrorLog "logs/error_log"
```
ログファイルの場所は以下の通りとなる
- /etc/httpd/logs/access_log
- /etc/httpd/logs/error_log


