<!--
title:   Visual StudioでC言語を学ぶための設定
tags:    C,VisualStudio,VisualStudio2022
id:      1e420180ff124cca605e
private: false
-->
:::note warn
C言語初心者向けの設定です。一部実務では好ましくない設定が含まれていますので、注意してください。
:::

プログラムを学ぶ際に最初に学ぶという人も多いかと思うC言語ですが、
Visual Studioで学びたいけどどのように設定すればいいかわからないという人のための記事です。

入門の書籍や学校などで学ぶ際のプログラムは、
Visual Studioで何も設定せずに実行するとエラーで弾かれてしまいます。
その辺を解消していきたいと思います。

:::note info
2022年4月現在で最新のVisual Studio 2022で解説します。
:::

# インストール

https://visualstudio.microsoft.com/ja/downloads/

より
コミュニティの下、
無料ダウンロードを選択してください。
※2022年4月時点のリンクです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/570881bf-ba81-d43f-1970-419666c35700.png)
自動でダウンロードが始まります。

::: note info
ブラウザの設定でダウンロード毎にフォルダー選択している場合は
フォルダー選択画面となりますので、適当なフォルダーへダウンロードしてください。
:::

ダウンロードした**VisualStudioSetup.exe**を実行。
下記画面はそのまま続行を選択。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/e6d47530-741a-562d-122e-cda7aa3bcc4d.png)
今回はC言語が使えればいいので、
**C++によるデスクトップ開発**のみにチェックを入れインストールを選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/53b45d9d-3f3f-d948-8630-9d1c36bd6bc2.png)

:::note info
他の言語とかもやってみたい人はここでやってみたいものにチェックをいれます。
:::

:::note warn
インストール後にも自由に追加・削除ができるので、
ここで必ずチェックを入れなければならないということはありません。
:::

:::note warn
他にもインストール先の指定とかの設定項目ありますが、設定したい人のみ設定してください。
:::


# 起動
インストール完了したら、Visual Studio 2022が起動します。
初回、Visual Studioにサインインの画面が表示されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/234e8b7c-d29e-032c-5053-815b82c949c5.png)

サインインを押すと、Microsoftアカウント(以降MSアカウント)の入力が求められますので、
MSアカウントを所持している場合は入力し、サインインしてください。
所持していない場合は**作成してください**よりアカウントの作成を行ってください。
**後で行う。** でMSアカウントを作成しなくても進めますが、
30日を過ぎると使用できなくなる制限版となってしまうため、
アカウントの作成は必須となります。

:::note info
アカウントの作成方法は省略します。表示される画面に沿って適宜作成してください。
:::

:::note info
後からでもMSアカウントのサインインはできますので(30日を過ぎてもOk)、ここでサインインしなかったからと言って、使えなくなるということはありません。
:::

サインイン後、配色テーマの選択画面になりますので、
見やすい配色を選択してください。

:::note info
すでにほかの端末などで1回でもサインインし設定を行っていた場合はその設定が引き継がれるため配色テーマの選択画面は省略されます。
:::

# 設定
### プロジェクトの作成
まず、プログラムを書くためにプロジェクトを作成します。
**新しいプロジェクトの作成**を選択してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/17faf191-0b28-3025-a41a-712495c34a4b.png)

一番上の**空のプロジェクト**を選択して次へ。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/6487da15-0dd1-0067-a371-813dce48dd26.png)

**新しいプロジェクトのを構成します**ではプロジェクト名とプロジェクトの保存先を指定します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/732f8759-6bb4-5d3b-82cc-9fd85edc8d59.png)


:::note info
プロジェクト名を変更すると一緒にソリューション名も変更されます。
ソリューション名は基本プロジェクト名と同じでいいかと思います。
:::

### プロジェクトの設定
まず最初にツールのオプションを選択し、オプションを表示させます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/afc48f70-0576-14dd-d8b9-1394884ed466.png)
そして、デバッグの全般を選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/47be68b4-04c0-d42f-798f-709ea9755352.png)
その中の**デバッグの停止時に自動的にコンソールを閉じる**にチェックを入れOKを選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/c57b83cf-0264-f865-cf3a-d4ef33b7b492.png)

:::note info
この設定項目は1度設定してしまえば設定変更しない限り、
Visual Studioを再起動しても残る設定となります。
:::

この項目を設定しないと、**～コード0で終了しました。** のエラーメッセージが表示されてしまいます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/ed0e476c-f8bd-b32d-e8f8-4d6965c6692e.png)


画面右端のソリューションエクスプローラーの中のソースファイルを右クリックして追加→新し項目を選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/79a63e3a-c367-5c47-2004-9aafad3e88af.png)
**C++ファイル**が選択されているのを確認し、
ファイル名を入力し、追加を選択します。

:::note warn
ファイル名の拡張子はデフォルトではcppですが、C言語の場合は.cに変更してください。
cppでもプログラムを動かすことはできますが、C++(C言語と互換性のある言語)用の拡張子のため、
C言語でプログラムする場合は明示的に拡張子を.cにしてください。
※今回はmain.cという名前に設定しています。
:::

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/2bd8f554-2804-c624-acf5-dc5f54029b81.png)


プログラムを書く際にソースコードの一番上に

```c
#define _CRT_SECURE_NO_WARNINGS
```

を記載します。
これは入門書などで使われる**scanf関数**などを使用できるようにするためです。

ただし、ソースコード上にわざわざ書きたくないという人は
プロジェクトのプロパティから
C/C++のプリプロセッサのプリプロセッサーの定義にて
**_CRT_SECURE_NO_WARNINGS**
を追加してください。

:::note warn
**_CRT_SECURE_NO_WARNINGS**はプロジェクト毎に設定する必要があります。
:::

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/45c70522-db56-93a5-bf60-66f2b0d79b7c.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/cd6fcd75-79c4-1194-2d44-90249983ae04.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/033f0daf-a56b-fb44-670e-95f814a70515.png)

:::note info
通常、scanfを使用することは非推奨のためプログラム実行時にエラーになってしまいますが、
これを抑制するために_CRT_SECURE_NO_WARNINGSを書く必要があります。
:::

:::note warn
Visual Studioでは**scanf関数**等の使用を推奨していません。
**scanf関数**の場合代わりに**scanf_s関数**を使用することを推奨しています。
実務で使用する際は**scanf関数**を使用するのは避けましょう。
:::

# おわりに
以上でVisual StudioでC言語でのプログラムを行うための環境構築が完了しました。
あとは書籍などを見てプログラムの勉強を行ってください。