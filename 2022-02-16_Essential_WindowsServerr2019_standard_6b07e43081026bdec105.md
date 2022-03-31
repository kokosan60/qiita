<!--
title:   Windows Server 2019 Essential→Standardアップグレードについて
tags:    Essential,WindowsServerr2019,standard,アップグレード
id:      6b07e43081026bdec105
private: false
-->
2019 EssentialからStandardにアップグレードの際に少しはまったので備忘録として記載します。

ネット検索したところ、

```bat
dism /online /set-edition:ServerStandard /accepteula /productkey:ProductKey
```

でアップグレードできるとのことだったので、
実行してみましたが、

「指定したプロダクトキーを検証できませんでした。
指定したプロダクトキーが有効であることと、ターゲットエディションに一致していることを確認してください。」

と表示されアップグレードができませんでした。
入力したのはMAKキーであり、正規のキーです。
なぜインストールできないのかとさらに調べていったところ、
それらしき回答が！！
[評価版からボリュームライセンスへの移行](https://social.technet.microsoft.com/Forums/ja-JP/5647e388-fe5d-41d7-abae-fee0b21a9f9b/35413203852925612363124251250812522125171254012512125211245212?forum=windowsserver2019ja)
MAKキーではなく、リテールまたはKMSキーで一度アップグレードしたのちにプロダクトキーの変更を行う必要があるとのこと。
[Key Management Services (KMS) client activation and product keys](https://docs.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys)
よりWindows Server 2019 StandardのKMSキーを取得、
コマンドのプロダクトキーのところにKMSキーを入力するとアップグレードが開始されました。
アップグレードには多少時間がかかるようなので放置して再起動
[Windows Serverのライセンス認証でコケる時](https://iranatilark.com/archives/2019/12/03-0102.php)
あとは
slmgr /ipk MAKキー
にてMAKキーに変更してあげればアップグレード完了。

※設定からのプロダクトキーの変更ではキー変更できなかったので注意！！