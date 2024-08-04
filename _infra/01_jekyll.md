---
permalink: /infra/jekyll
layout: single
title: "How to use Jekyll"
---

## 環境の前提

* Windows 11
* WSL2

## Jekyll Install

{% highlight bash %}
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y ruby-full build-essential zlib1g-dev

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install bundler

gem install jekyll

jekyll --version
{% endhighlight %}

## Use minimal mistakes theme

下記を参考に。  
[Quick Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#starting-from-jekyll-new)

{% highlight bash %}
jekyll new githubpages
cd githubpages
{% endhighlight %}
* _config.yml の編集
  * 下記URLからコピペしてくる  
[_config.yml](https://github.com/mmistakes/minimal-mistakes/blob/master/_config.yml)
  * なぜかTheme 部分がコメントアウトされているのでコメントインする
* index.html の編集
下記にする  
[index.html](https://github.com/mmistakes/minimal-mistakes/blob/master/index.html)
* デフォルトで作成される_post配下のエントリのLayoutを single に帰る
* about.md を削除
* ある程度できたらjekyll serve コマンドでローカルホストで起動し状況確認

## Other customize

### author 情報の表示をFalseにする
* _config.yml のauthor-profile をfalse にする

### 上のバーをカスタマイズする
* _data/navigator.yml を下記に。
{% highlight bash %}
main:
  - title: "Blog"
    url: /blogs/
  - title: "About"
    url: /about/
  - title: "Infra"
    url: /infra/jekyll
{% endhighlight %}
### infra ページを作る
* _infra ディレクトリを作る
* _config.yml に_infraを読み込んでくれるように下記設定を追記
{% highlight bash %}
collections:
  infra:
    output: true
{% endhighlight %}
* _config.yml にside bar が出るように下記設定を追加
{% highlight bash %}
  # _infra
  - scope:
      path: ""
      type: infra
    values:
      layout: single
      sidebar:
        nav: "infra"
{% endhighlight %}
* side bar 定義：　_data/navigotor.yml 下記追加
{% highlight bash %}
infra:
  - title: Web
    children:
      - title: "How to start jekyll"
        url: /infra/jekyll

  - title: Linux
    children:
      - title: "Sample"
        url: /linux/sample/

  - title: Security
    children:
      - title: "Sample2"
        url: /security/sample2/
{% endhighlight %}





