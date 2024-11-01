---
permalink: /infra/windowsclient
layout: single
title: "Windows 11のバックアップ方針"
---

## 概要

* Windows Client においてバックアップの方式をここに記す。
* 新しいPC をセットアップ時にも簡単にできるよう現在の設定を記載することにより、今後メインPCの故障時にも即座に原状復帰できるようにする

## やり方

* Onedrive はWindows Client をインストール時に基本入っている。
そのためOneDrive に最初に必要なスクリプト類や情報のポイント先を入れておくのがベターである。
* 下記で対応をしてみたが、winget コマンドがなぜか繋がらずうまくいかなくて断念
* Chocolatey をインストール
{% highlight powershell %}
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
{% endhighlight %}
* Boxstarter をインストール
{% highlight powershell %}
choco install Boxstarter
{% endhighlight %}
* Boxstarter で設定を実施
{% highlight powershell %}
BOXSTARTERSHELL
Install-BoxstarterPackage -PackageName {file path}
{% endhighlight %}
* Boxstarter の設定は下記
{% highlight powershell %}
# Boxstarterとwingetを組み合わせたサンプルスクリプト

#--- セットアップ用の一時的な設定 ---
Update-ExecutionPolicy Unrestricted
Disable-UAC

#--- Windows Settings ---
Disable-GameBarTips
Disable-InternetExplorerESC
Set-WindowsExplorerOptions -EnableShowHiddenFilesFoldersDrives -EnableShowFileExtensions
Set-TaskbarOptions -Size Small -Dock Bottom -Combine Full -Lock
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced -Name NavPaneShowAllFolders -Value 1 # all file show

#--- Wingetによるアプリインストール ---
# wingetをPowerShellから呼び出してアプリをインストール
$packageListPath = "packages.txt" 
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
* Google IME のKeymap 変更  
Keymap.txt をExport しておいておくのでこれをImportする
* WinAuth のデータ移行
  * Export 機能があるのでデータをExport して移行する
* データ復元系