---
layout: post
title: "fluentdをEC2にいれてapacheのログをMondoDBに残していく"
date: 2013-06-05 20:19
comments: true
categories: [fluentd]
sharing: true
footer: true
keywords: [fluentd]
---
fluentdを以下のような構成で使ったときのメモ。   
apacheのaccess_logを収集する場合。

![configuration](/images/2013-06-06/fluentd.png) 

## やったことリスト
 * application server側
   * fluentd install
   * /etc/td-agent/td-agent.conf設定
 * log server側
   * MongoDB install
   * fluentd install
   * /etc/td-agent/td-agent.conf設定

## やったこと詳しく

### application server側

 * fluentd install

```sh
$ ssh -i ec2.pem ec2-user@ec2.com
## 日本時間に合わせる(不要な人は不要)
$ sudo cp /usr/share/zoneinfo/Japan /etc/localtime
## install
$ curl -L http://toolbelt.treasure-data.com/sh/install-redhat.sh | sh
### plugin install
## hostname をlog server側に渡したいので必要だったplugin
$ sudo /usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-config-expander --no-ri --no-rdoc
$ sudo /etc/init.d/td-agent start
$ sudo /etc/init.d/td-agent status

### test !
## defaultのconfigに8888番が割り当てられている
$ curl -X POST -d ‘json={“json”:”test”}’ http://localhost:8888/debug.test
## ログ確認 !
$ tail -1 /var/log/td-agent/td-agent.log
2013-06-03 11:11:11 +0900 debug.test: {"json":"message"}
```

 * /etc/td-agent/td-agent.conf設定

```sh
### 権限グループ変更
## これをやっておかないと起動後に怒られる
$ sudo chgrp td-agent /var/log/httpd/
$ sudo chmod g+rx /var/log/httpd/
### 設定ファイル変更
$ vi /etc/td-agent/td-agetn.conf
$ cat /etc/td-agent/td-agetn.conf
## 図中 *1 のための設定
<source>
  type config_expander
  <config>
    type tail
    format /^(?<${hostname}>.*)$/
    path /var/log/httpd/access_log
    ## ログをどこまで送ったかのポジションを覚えているファイルの設定
    pos_file /var/log/td-agent/access_log.pos
    tag apache.access_log
  </config>
</source>
 
## 図中 *2 のための設定
<match apache.access_log>
  type forward
  flush_interval 5s
  ## ログを送っロウとしたときに遅れなかった場合のバッファを指定するための設定
  ## デフォルトだとメモリなので設定が必要
  buffer_type file
  buffer_path /var/log/td-agent/buffer/access_log
 
  <server>
    host ec2.com # EC2のpublic DNS
    port 24224
  </server>
</match>
```

### log server側

 * MongoDB install

```sh
$ sudo vi /etc/yum.repos.d/10gen.repo
$ cat /etc/yum.repos.d/10gen.repo
[10gen]
name=10gen Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64
gpgcheck=0
enabled=1
$ sudo yum install mongo-10gen-server mongo-10gen -y
$ sudo chkconfig mongod on
$ sudo /etc/init.d/mongod start
$ sudo /etc/init.d/mongod status
$ mongo
> show dbs
```

 * fluentd install
   * application server側のfluentd installと一緒
 * /etc/td-agent/td-agent.conf設定

```sh
$ sudo vi /etc/td-agent/td-agent.conf
$ cat /etc/td-agent/td-agent.conf
## 図中 *3 のための設定
<source>
  type forward
  port 24224
</source>
  
<match apache.access_log>
  type mongo
  database apache
  collection access_log
  
  host localhost
  port 27017
  
  flush_interval 10s
</match>
```
  * ポート解放
    * AWSコンソール上などからtcp/udpの27017を解放してあげる

### 動作確認
 * application serverにブラウザなどからアクセスする
 * log serverに入ってmongo queryで確認

```sh
$ mongo apache
> db.access_log.find({'time':{$gt:ISODate("2013-06-03T13:13:13Z")}})
{ "_id" : ObjectId("5191c59a2e3d4172a30017e2"), "ip-10-10-10-10" : "x-forwarded-for:\"-\"\thost:20.20.20.20\tident:-\tuser:-\ttime:[03/July/2013:13:03:03 +0900]\trequest-first-line:\"GET / HTTP/1.1\"\tstatus:200\tresponse-size:-\trefer:\"-\"\tuser-agent:\"USER-AGENT\"\trequest-time:6666", "time" : ISODate("2013-06-03T04:03:03Z") }
```

