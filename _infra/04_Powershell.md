---
permalink: /infra/powershell
layout: single
title: "Powershell Tips"
---

## Powershell 内でVim を使う

Vim がないと生きていけないのでまずはインストール
* インストール先：[Downloading Vim](https://www.vim.org/download.php)
* $profile コマンドをPowershell 上で実行し、ファイルを見つける
* 対象ファイルを開いて下記を追記 (ファイルが存在しなければ自分でディレクトリ含めて作る)
{% highlight bash %}
set-alias vi "C:\Program Files\Vim\vim91\vim.exe"
set-alias vim "C:\Program Files\Vim\vim91\vim.exe"
{% endhighlight %}

## Powershell 内 Vim での操作

* コピペはCtrl +Shift + v/c

## 引数について

* 引数が4つあるものについて3つ目の引数に値をいれたいとき
{% highlight powershell %}
Param(
    [String]$Arg1,
    [Int]$Arg2,
    [Switch]$Arg3,
    [String]$Arg4
        )
Write-Host $Arg1
Write-Host $Arg2
Write-Host $Arg3
Write-Host $Arg4
{% endhighlight %}
  * 実行結果
{% highlight powershell %}
> powershell .\test.ps1 -Arg4 test

0
False
test
{% endhighlight %}
* 問題は、この3つ目の引数にTrue を入れようとするとこんな事象が起きる
{% highlight powershell %}
> powershell .\test.ps1 -Arg3 $true
True
0
True
{% endhighlight %}
  * switch をTrue にしたいときは、その値を指定してやるだけでいい
{% highlight powershell %}
> powershell .\test.ps1 -Arg3

0
True
{% endhighlight %}