---
permalink: /infra/certificate
layout: single
title: "Apache Certificate"
---

## 環境の前提

* Rocky Linux release 8.10 (Green Obsidian)
* OpenSSL 1.1.1k  FIPS 25 Mar 2021

## やりたいこと

* Apache でクライアント認証をさせる(割愛。どこかで書く)
* Apache でHTTPS サーバ認証でWarning を最小限にする

## オレオレサーバ証明書の作成方法
* 下記のサイトが詳しいのでこちらのとおりに実施でできた  
[オレオレ証明書をopensslで作る（詳細版）](https://ozuma.hatenablog.jp/entry/20130511/1368284304)
{% highlight bash %}
$ openssl genrsa 2048 > server.key
$ openssl req -new -key server.key > server.csr
$ openssl x509 -days 3650 -req -signkey server.key < server.csr > server.crt
{% endhighlight %}
* 確認コマンド類は下記
{% highlight bash %}
openssl rsa -text < server.key
openssl req -text < server.csr
openssl x509 -text < server.crt
{% endhighlight %}
* Chrome でエラーを最小限にするにはSANの設定をしてやるといい

## その他

* 昔は証明書にEV とか色々ランクがあって、いいものだとブラウザが緑色になってよかったねとかがあったのだけど、最近ないなぁと思って調べてみた
* 下記の記事が詳しいが、結論としては緑色バーにしてみたけども、偽造することができるしブラウザの領域を無駄に使うしこの機能意味ないのでは？ということ  
[Google Chrome EV表示の終焉](https://jovi0608.hatenablog.com/entry/2019/08/12/095854)  
[TLSとWeb～URLブラウザの表示のいまとこれからバーの表示はどうなるのか～](https://www.nic.ad.jp/sc-sendai/program/iwsc-sendai-d2-5.pdf)