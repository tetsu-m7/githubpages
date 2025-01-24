---
layout: single
title:  "IP Geolocation について"
---

# はじめに

IP Geolocation についてある程度わかっていることをまとめます。  

# そもそもIP Geolocation とは？

IP Geolocation は、IPアドレスに対してそのIPがどこの地域に所属しているかを示しているものです。  
現在様々な会社でIP Geolocation のDatabase は公開されておりその精度は様々です。  
よく使うツールとしては下記があります。これらのツールでも表示される国名がそもそも違うことも多いです。  
* [IPひろば](https://www.iphiroba.jp/ip.php)
* [IP Void](https://www.ipvoid.com/)
* [VirusTotal](https://www.virustotal.com/gui/)

また、その他セキュリティベンダやSIEM ツール自体もIP Geolocation の情報を持っている事があり、利用されるツールによって情報が違ったりします（Paloalto やFortigate など）。  
確認する限り IP Geolocation の特定方法はまずはIP アドレスに対する逆引きでそのOwner 情報をある程度判断し、利用ISP などを経路情報から特定し、場所を特定しているのだと思われます。  
IP アドレスは常に譲渡などで所属が変わる可能性があるため、自分で場所情報を推定するのは大変なため、一般的にはVendor 情報を利用することが多いでしょう。

# 海外ローミングについて

例えば日本の携帯をアメリカなどの海外に持っていったときに、そのIPアドレスはどうなるでしょうか？  
答えは **日本のIPアドレス** になります。  

ネットワークに詳しい人であれば海外に持っていったらAT&T などの海外基地局につながっているわけなので、海外のIPアドレスになるのでは？と思うかもしれません。  
実は私もそうだったのですがこちらは海外の基地局から日本に戻ってからインターネットに行くようで、日本のIPアドレスになるようです。  
下記にDocomo の解説があります。

* https://www.docomo.ne.jp/service/world/roaming/check/dataroaming/

# Starlink 衛星通信について

これが今回メモを書こうと思った理由です。Starlink についてです。  
最近SANSで下記のような記事が出ました  
* [Geolocation and Starlink](https://isc.sans.edu/diary/rss/31612)

ざっくり内容を簡単に書くと次のような話になります。  
* 衛星によるネットワークがStarlink により普及してきて一般的になってきている
* Starlink は他の衛星ネットワークとは違い低軌道の衛生を利用している
* そのため通信する衛生はこれまでの衛生とは違い比較的狭い範囲の基地局に電波を送信することになり、地理的な制約を受ける
* Starlink のIPをブロックしているWeb サイトが一定数あり、ユーザは影響を受けている

Starlink 自体のネットワーク経路は下記にも詳しく記載があり参考になります。　　
[話題の衛星インターネット「Starlink」の速度や経路を技術者視点で調べてみた](https://atmarkit.itmedia.co.jp/ait/articles/2302/17/news007.html)


特に興味深い点として、Starlinkは衛星通信でありながら、ある程度Geolocationに依存したIPアドレスを持つということが挙げられます。　　
日本のStarlinkのIPアドレスは65.181.4.0/22付近のようで、この範囲を調べるとGeolocationの情報がある程度確認できます。　　
衛星通信ではGeolocationがあまり信頼できないものと思っていましたが、こうした技術的な調査を通じて一定の根拠を得られた点が、とても面白いと感じました。


