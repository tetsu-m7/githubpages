---
permalink: /infra/rundeck
layout: single
title: "Rundeck"
---

## 環境の前提

* Rocky Linux release 8.10 (Green Obsidian)

## Rundeck のインストール

* 公式ドキュメント
  * [Installing on Red Hat, Amazon or Oracle Linux distributions](
https://docs.rundeck.com/docs/administration/install/linux-rpm.html)

* Install 方法
{% highlight bash %}
sudo yum install java-11-openjdk-devel
curl https://raw.githubusercontent.com/rundeck/packaging/main/scripts/rpm-setup.sh 2> /dev/null | sudo bash -s rundeck
dnf install rundeck
service rundeckd start
{% endhighlight %}

* IPv6のDisableをしていないと、IPv6で起動してしまうときがある。
{% highlight bash %}
vim /etc/default/grub
下記を加える
GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX ipv6.disable=1"
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-mkconfig -o /boot/efi/EFI/rocky/grub2.cfg
reboot
{% endhighlight %}

* firewalld がデフォルトで通信を止めている可能性がある。テストであれば一旦firewalldを止めておく
{% highlight bash %}
systemctl stop firewalld
{% endhighlight %}

* URL がlocalhostではなく一旦IPなのでその値に変える
  * /etc/rundeck/rundeck-config.properties を開き、下記に変更
{% highlight bash %}
grails.serverURL=http://192.168.231.129:4440
{% endhighlight %}

* ログインする。(今回はVMなので自分のVMのIPに変える)
  * http://192.168.231.129:4440/ in a browser
