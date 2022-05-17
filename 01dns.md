# DNSサーバー構築
## DNSサーバのインストール
yum install bind  
- 強制的にyにする方法は？  
yum -y install bind
- アンインストール  
yum remove bind
## DNSサーバの起動と終了
- 起動  
systemctl start named
- 状態  
systemctl status named  
Active: active (running) になっている
- 終了  
systemctl stop named  
Active: inactive (dead)になっている
- 自動起動の状態  
systemctl is-enabled named  
disabledは停止
- 自動起動をオンにする
systemctl enable named  
systemctl is-enabled named で確認すると、enabledになっている
- 自動起動をオフにする
systemctl disable named  
## FireWallの設定
- 状態  
systemctl status firewalld
- public zoneで許可したサービスとポートを確認する  
firewall-cmd --list-all --zone=public
- ルールを再起動しても(恒久的に)適用させたい場合には --permanent オプションを付ける  
firewall-cmd --add-service=dns --zone=public --permanent  
firewall-cmd --reload
## /etc/named.confの編集
- DNSSECの停止  
dnssec-enable no;  
dnssec-validation no;  
>** VIエディタの文字列検索  
/ 検索文字列
- /etc/named.confの一部を以下のように編集
```  
recursion yes;

forward only;
forwarders { 192.168.3.1; };

dnssec-enable no;
dnssec-validation no;

empty-zones-enable no;
```
## /etc/sysconfig/namedの末尾に追加
```
OPTION="-4"
```

```
どこか間違っている  
Job for named.service failed because the control process exited with error code. See "systemctl status named.service" and "journalctl -xe" for details.
```
## さらにnamed.confを編集  
- listen-onにDNSサーバのIPアドレスを追加
- allow-query をanyに変更
- zone ステートメントを編集　exmaple.jpに対する正引き、逆引きのゾーンファイルを設定
```
options {
        listen-on port 53 { 127.0.0.1; 192.168.64.3;};
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { any; };
…中略
```
```
zone "example.jp" IN {
        type master;
        file "example.zone";
};

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "rev.zone";
};
```
## 正引きゾーンファイルを編集する  
/var/namedにあるnamed.emptyをコピーして、example.zoneを作成
```
$TTL 3H
@       IN SOA  ns.example.jp. nsadmin.example.jp. (
                                        2022050401      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
ns      A       192.168.64.3
www     A       192.168.64.3
```
>** ゾーンファイルの中身  
https://atmarkit.itmedia.co.jp/fnetwork/dnstips/014.html

## 逆引きゾーンファイルを編集する  
/var/namedにあるnamed.emptyをコピーして、rev.zoneを作成
```
$TTL 3H
@       IN SOA  ns.example.jp. nsadmin.example.jp. (
                                        2022050401      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns
ns      A       192.168.64.3
www     A       192.168.64.3                              
```
>** 逆引きについて  
https://atmarkit.itmedia.co.jp/fnetwork/dnstips/016.html

## named.conf文法チェック  
```
named-checkconf 
named-checkzone example.jp /var/named/example.zone 
zone example.jp/IN: loaded serial 2022050401
OK
named-checkzone 64.168.192.in-addr.arpa /var/named/rev.zone
zone 64.168.192.in-addr.arpa/IN: loaded serial 2022050401
OK
```
## 動作チェック
- 再読込  
systemctl reload named
- 再起動（必要ないが）
systemctl restart named
- 名前解決確認
```
nslookup www.example.jp 192.168.64.3
Server:		192.168.64.3
Address:	192.168.64.3#53

Name:	www.example.jp
Address: 192.168.64.3
```
> nslookupがない場合  
yum install bind-utils  
もしくは　yum groupinstall base

> permmissionが許可されていないと以下のようなエラーとなる　　
```
systemctl status named -l
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
   Active: active (running) since 水 2022-05-04 13:07:01 JST; 26min ago
 Main PID: 16770 (named)
   CGroup: /system.slice/named.service
           └─16770 /usr/sbin/named -u named -c /etc/named.conf
 5月 04 13:07:01 localhost.localdomain named[16770]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
 5月 04 13:07:01 localhost.localdomain named[16770]: zone example.jp/IN: loading from master file example.zone failed: permission denied
 5月 04 13:07:01 localhost.localdomain named[16770]: zone example.jp/IN: not loaded due to errors.
 5月 04 13:07:01 localhost.localdomain named[16770]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa/IN: loaded serial 0
 5月 04 13:07:01 localhost.localdomain named[16770]: zone localhost/IN: loaded serial 0
 5月 04 13:07:01 localhost.localdomain named[16770]: zone localhost.localdomain/IN: loaded serial 0
 5月 04 13:07:01 localhost.localdomain named[16770]: all zones loaded
 5月 04 13:07:01 localhost.localdomain named[16770]: running
 5月 04 13:07:01 localhost.localdomain systemd[1]: Started Berkeley Internet Name Domain (DNS).
 5月 04 13:07:01 localhost.localdomain named[16770]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
 ```
 以下、何が原因か分かりますか？example.zoneのグループが違っていますね？？
 ```
[root@localhost named]# ls -l
合計 24
drwxrwx---. 2 named named   23  5月  4 10:56 data
drwxrwx---. 2 named named   60  5月  4 13:07 dynamic
-rw-r-----. 1 root  root   194  5月  4 13:03 example.zone
-rw-r-----. 1 root  named 2253  4月  5  2018 named.ca
-rw-r-----. 1 root  named  152 12月 15  2009 named.empty
-rw-r-----. 1 root  named  152  6月 21  2007 named.localhost
-rw-r-----. 1 root  named  168 12月 15  2009 named.loopback
-rw-r-----. 1 root  root   216  5月  4 13:00 rev.zone
drwxrwx---. 2 named named    6  2月 24 02:17 slaves
 ```

