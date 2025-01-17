---
permalink: /infra/windowsclient
layout: single
title: "Windows 11のバックアップ方針"
---

## 概要

* Windows Client においてバックアップの方式をここに記す。
* 新しいPC をセットアップ時にも簡単にできるよう現在の設定を記載することにより、今後メインPCの故障時にも即座に原状復帰できるようにする

## やり方

* PowerToys で境界線のないマウスを入れたうえで情報連携すると比較的簡単
* Chocolatey をインストール
{% highlight powershell %}
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
{% endhighlight %}
* Boxstarter をインストール
{% highlight powershell %}
choco install Boxstarter
{% endhighlight %}
* 設定ファイルを事前にDesktop かどこかにおいておく。その後Boxstartershell を実行し、コマンドで設定を実施
{% highlight powershell %}
BOXSTARTERSHELL
Install-BoxstarterPackage -PackageName {setting file path}
{% endhighlight %}
* Boxstarter の設定は下記
{% highlight powershell %}
# Boxstarterとwingetを組み合わせたサンプルスクリプト

#--- セットアップ用の一時的な設定 ---
Update-ExecutionPolicy Unrestricted
Disable-UAC

#--- Windows Settings ---
Disable-GameBarTips
Set-WindowsExplorerOptions -EnableShowHiddenFilesFoldersDrives -EnableShowFileExtensions
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced -Name HideFileExt -Value 0 # all file show

#--- Wingetによるアプリインストール ---
# wingetをPowerShellから呼び出してアプリをインストール
$packageListPath = "C:\Users\xxxx\Desktop\package.txt" 
Get-Content $packageListPath | ForEach-Object {
    # 各行のパッケージIDでwingetを呼び出してインストール
    Invoke-Expression "winget install --id $_ -e --silent" 
}

#--- タスクバーへのアプリピン留め（必要に応じて）---
Install-ChocolateyPinnedTaskBarItem "$env:windir\system32\mstsc.exe" 

#--- Restore Temporary Settings ---
Enable-UAC
Enable-MicrosoftUpdate
Install-WindowsUpdate -acceptEula

{% endhighlight %}

その他細々系
* Ctrl2Cap  
[Ctrl2Cap](https://learn.microsoft.com/en-us/sysinternals/downloads/ctrl2cap)
* Windows HomeはRDP接続ができないのでChrome remote desktop を入れておく
* WinAuth のデータ移行
  * Export 機能があるのでデータをExport して移行する
* [alt-ime-ahk](https://github.com/karakaram/alt-ime-ahk)
* Client 証明書をNASからいれる
* Powershell で wsl --install を叩く
* Boxstarter Chocolate の削除：下記ディレクトリの削除
C:\ProgramData\chocolatey   
C:\ProgramData\ChocolateyHttpCache   
C:\ProgramData\Boxstarter   
* mkdir linux_home on C:\ then vim /etc/passwd to change linux_home
