# dnsmasq_poc

## description
dnsmasqを使ってローカルでDNSの挙動を確かめるときに使えそうなやーつ

## how to use

初期状態
```bash
$ docker compose up -d
% dig @localhost current.db.test CNAME                                             

; <<>> DiG 9.10.6 <<>> @localhost current.db.test CNAME
; (3 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9486
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;current.db.test.               IN      CNAME

;; ANSWER SECTION:
current.db.test.        600     IN      CNAME   v2.db.test.

;; Query time: 11 msec
;; SERVER: ::1#53(::1)
;; WHEN: Sat Oct 05 19:53:52 JST 2024
;; MSG SIZE  rcvd: 68
```

currentを書き換える(ホストPCからでOK)
```bash
% diff dnsmasq.conf.bak dnsmasq.conf
% sed -i.bak 's/cname=current\.db\.test,v2\.db\.test/cname=current.db.test,v3.db.test/' dnsmasq.conf
% diff dnsmasq.conf.bak dnsmasq.conf
16c16
< cname=current.db.test,v2.db.test
---
> cname=current.db.test,v3.db.test
```

再起動
```bash
$ docker compose restart dnsmasq
```

digで確認. v3.db.testに変更されてる
```bash
% dig @localhost current.db.test CNAME

; <<>> DiG 9.10.6 <<>> @localhost current.db.test CNAME
; (3 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55178
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;current.db.test.               IN      CNAME

;; ANSWER SECTION:
current.db.test.        600     IN      CNAME   v3.db.test.

;; Query time: 9 msec
;; SERVER: ::1#53(::1)
;; WHEN: Sat Oct 05 20:06:55 JST 2024
;; MSG SIZE  rcvd: 68
```

## DNS解決がうまくいかない場合

ログ見よう
```bash
% docker compose exec dnsmasq cat /var/log/dnsmasq.log
Oct  5 10:45:12 dnsmasq[1]: started, version 2.78 cache disabled
Oct  5 10:45:12 dnsmasq[1]: compile time options: IPv6 GNU-getopt no-DBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth no-DNSSEC loop-detect inotify
Oct  5 10:45:12 dnsmasq[1]: reading /etc/resolv.conf
Oct  5 10:45:12 dnsmasq[1]: using nameserver 127.0.0.11#53
Oct  5 10:45:12 dnsmasq[1]: read /etc/hosts - 7 addresses
Oct  5 10:45:17 dnsmasq[1]: query[CNAME] current.db.test from xxx.xxx.xxx.xxx
Oct  5 10:45:17 dnsmasq[1]: config current.db.test is <CNAME>
```