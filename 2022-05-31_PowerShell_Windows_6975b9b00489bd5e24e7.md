<!--
title:   PowerShellを使ってソフトウェアの最新版を自動ダウンロード、インストールする
tags:    PowerShell,Windows,ソフトウェア
id:      6975b9b00489bd5e24e7
private: false
-->
ソフトウェアをインストールするときに、
いちいちサイトに行って最新版を探して落としてこなければならないのが面倒くさい。

そこで、サイトから自動で最新版のソフトウェアを自動でダウンロードするスクリプトをPowerShellで組んでみました。

### 必須条件
* インストーラーの直リンクが判明していること

今回使うコマンドでは直リンクからのダウンロードを行うため、
サイト上のスクリプトか何かでリダイレクトされている場合などは対応していません。

# スクリプト

今回は例として、LhaplusとFFFTPを自動ダウンロード、インストールするスクリプトを記載します。

```powershell:lhaplus
#インストーラーをダウンロードするためのフォルダー(適時変更)
$dir="C:\temp"

 if(-not (Test-Path $dir)){
    New-Item $dir -ItemType Directory
}

#IEの初期設定が完了していない場合、-UseBasicParsingをつけないとエラーとなる
#指定したWebページのソースを取得
$source = Invoke-WebRequest "http://hoehoe.com/" -UseBasicParsing

#Lhaplusのサイトはそのままでは文字化けしてしまっているため、Shift_JISへ変換
#海外サイトとかでは変換の必要なし
$source_jis = [System.Text.Encoding]::GetEncoding("Shift_JIS").GetString( [System.Text.Encoding]::GetEncoding("ISO-8859-1").GetBytes($source.Content) )

#正規表現にてソースから最新バージョン情報を拾う
#1つ以上の絵連続した数字+.+2つの連続した数字で合致する個所を取得
#2022年6月現在では1.74
$version = (([regex]::match($source_jis,'\d+\.\d{2}')).value).replace(".","")

#直リンクよりダウンロード(先ほど取得したバージョン情報が必要)
Invoke-WebRequest -Uri "http://www7a.biglobe.ne.jp/~schezo/lpls$($version).exe" -outfile "$($dir)\lhaplus.exe"

#サイレントインストール
Start-Process -FilePath "$($dir)\lhaplus.exe" -ArgumentList "/VERYSILENT /NORESTART" -Wait

#インストーラーをダウンロードするためのフォルダーごと削除(もともとあったファイルも削除されてしまうため要注意!!)
Remove-Item $dir -Recurse -force
```

```powershell:FFFTP
$dir="c:\temp"

 if(-not (Test-Path $dir)){
    New-Item $dir -ItemType Directory
}
$source = Invoke-WebRequest "https://github.com/ffftp/ffftp/releases/" -UseBasicParsing

# リリースページから最新版を取得
$installer = ("https://github.com" + [regex]::match($source.content,"/ffftp/ffftp/releases/download/v(\d+\.)+\d/ffftp-v(\d+\.)+\d-x64.msi"))

Invoke-WebRequest -uri $installer -OutFile ($dir + "\ffftp-x64.msi")

$installer_path = ($dir + "\ffftp-x64.msi")

Start-Process -FilePath msiexec -ArgumentList "/i $installer_path /quiet /norestart" -Wait

Remove-Item ($dir + "\ffftp-x64.msi") -force
```

# 作成方法

作成方法としては、

* インストーラーのダウンロードページにアクセス
* インストーラーの直リンクがあること確認
* 直リンクにバージョン情報がある場合は、
* ダウンロードページより正規表現で拾えそうなバージョン情報を見つけ、
* バージョン情報と組み合わせて直リンクとなるようにURLを生成
* サイレントインストール可能か調べる

以上を基としてPowerShellを作成します。

※今回、正規表現の所をmatchで検索していますが、
通常一番上に新しいバージョンが記載されるため、
matchを使用しています(matchは最初にマッチしたもののみ抽出)。
マッチしたものすべてを拾う場合はmatchesに変更する必要があります。
サイトによって使い分けてください。

今回、作成するにあたり**Invoke-WebRequest**コマンドレッドを使用しています。
このコマンドレッドにてWebページのソースの取得、インストーラーのダウンロードを行っています。

サイトによってはソースコードをエンコードしなければならない場合があるので要注意です。

-UseBasicParsingオプションを付けることで、IEの初回設定を行っていない場合でも正常にソース取得できるようになります。
※オプション付けないとサイトによってはIEのエンジンを用いて処理するため、初回設定が終わっていないとエラーになってしまいます。
ですので、このオプションは常につけておいた方がいいかと思います。

サイレントインストールについてはググれば基本的に情報が載っているので、参考にしてください。
下記コマンドにてサイレントインストール実施しています。
`powershell
Start-Process -Filepath インストーラーの絶対パス -ArgumentList サイレントインストール用オプション -wait
`
-waitオプションを指定することでインストールが終わるまで次のコマンドを実行しない状態となります。