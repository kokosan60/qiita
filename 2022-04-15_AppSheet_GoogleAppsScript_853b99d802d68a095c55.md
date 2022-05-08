<!--
title:   バーコード蔵書管理アプリに手を加えてみた
tags:    AppSheet,GoogleAppsScript,書籍
id:      853b99d802d68a095c55
private: false
-->
[AppSheetと20行のGoogle Apps Scriptで作るバーコード蔵書管理アプリ](https://qiita.com/takatama/items/fa141cb13ccc7324f887)
が大変すばらしかったので、自分でも実装してみましたが、

この記事で使用している  [OpenBD API](https://openbd.jp/#first)では対応していない書籍がいくつか見られたので、
見つからなかった場合他のAPIより取得という機能を追加してみました。

:::note info
スプレットシートのフォーマットは[元記事](https://qiita.com/takatama/items/fa141cb13ccc7324f887)のをそのまま使用しています。
:::

:::note warn
他のAPIではPubDateの表示形式が異なっていたため、そのままで設定すると一部表示形式が異なって表示されてしまうので、
PubDateの列全体の表示形式をスプレットシート上であらかじめ**yyyy/mm/dd**形式に変更しておく必要があります。
:::

# GoogleAppsScriptについて
OpenBDのAPIでヒットしなかったまたはヒットしたけど著者が空欄だったら楽天のAPIを、
楽天のAPIでヒットしなかったらGoogleのAPIを検索し、
GoogleのAPIでもヒットしなかったらIDとCreateAtの列だけ追記します。
※一部書籍について、OpenDBのAPIでは書籍の画像がヒットしなかったため、画像がヒットしない場合は画像だけGoogleのAPIのものを使用しています。
※念のため楽天のAPIについても同様に画像はヒットしなければGoogleのものを使用するコードを記載しています。
※楽天のPubDateは日にちまでの記載がなかったり(**例：2016年03月**)、**2016年03月04日頃**とかの曖昧な記載が多かったので、Googleのものに変更しています。
※楽天のアプリIDは[**こちら**](https://webservice.rakuten.co.jp/app/create)より登録することで取得できます（要楽天ID、URLは適当なものでOK）。
※スプレッドシート上にISBN直打ちでも動作するように**ID**と**CreateAt**は変更しています。
※ISBNの重複は避けたいので、新規登録時に重複していた場合はエラーメッセージを表示させ、該当ISBNの行を削除しています。

以下コード

:::note warn
GASは始めたばかりなので、コード見ずらい部分あるかと思いますが、ご容赦ください。
:::

```javascript

/* 2022-04-22 楽天で画像見つからなかったときにGoogleのAPIを取ってこなかったバグ修正、
              Googleでも画像が見つからなかった時の処理追加
              重複の際のデーター削除時、
　　　　　　　 削除列の一つ下のセルがアクティブな状態で止まっていたのを、
　　　　　　　 削除列のセルでアクティブになるよう変更
　　　　　　　 それに伴い、スプレットシートIDの記述削除 */
/* 2022-05-03 著作者情報がない書籍の場合エラーになっていたのを修正 */
/* 2022-05-03 漫画において巻数が表示されなかったため表示されるよう変更(OpenBDのみ対応) */

const rakuten_appid= /*楽天のアプリIDを記載*/

function fetchBookSummary_openbd(isbn) {
  const url = 'https://api.openbd.jp/v1/get?isbn=' + isbn;
  const res = UrlFetchApp.fetch(url,{muteHttpExceptions: true});
  const json = JSON.parse(res.getContentText());
  if (!json[0]) return null;
  return json[0];
}

function fetchBookSummary_rakuten(isbn) {
  const url = 'https://app.rakuten.co.jp/services/api/BooksBook/Search/20170404?format=json&isbn=' + isbn+ '&applicationId=' + rakuten_appid ;
  const res = UrlFetchApp.fetch(url,{muteHttpExceptions: true});
  const json = JSON.parse(res.getContentText());
  if(json["totalItems"] == 0) return null;
  if(json["Items"].length == 0) return null;
  return json["Items"][0];
}

function fetchBookSummary_google(isbn) {
  const url = 'https://www.googleapis.com/books/v1/volumes?q=isbn:' + isbn + "&country=JP";
  const res = UrlFetchApp.fetch(url,{muteHttpExceptions: true});
  const json = JSON.parse(res.getContentText());
  if(json["totalItems"] == 0) return null;
  if(json["items"].length == 0) return null;
  return json["items"][0];
}

function getDate(strDate) {
  return new Date(strDate.slice(0, 4), strDate.slice(4, 6), strDate.slice(6, 8));
}

function getNowDate(){
  let d = new Date();
  return Utilities.formatDate(d, "Asia/Tokyo", "yyyy/MM/dd HH:mm:ss");
}

function onChangeSheet(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  sheet.getDataRange().getValues().forEach((row, i) => {
    const isbn = row[1], title = row[2];
    if (isbn && title) return; //すでに登録済みの場合は飛ばす
    let isbn_list = sheet.getRange(2,2,sheet.getLastRow()-1).getValues();
    let count=isbn_list.length;
    let cnt=0;
    for(let j=0;j<count;j++){
        if(isbn == isbn_list[j][0]){
          cnt++;
        }
    }
    if(cnt>1){
      Browser.msgBox("エラー!!","ISBNが重複しています。", Browser.Buttons.OK);
      const lrow = sheet.getLastRow();
      sheet.deleteRow(lrow);
      sheet.getRange(lrow,2).activate();
      return;
    }
    //OpenBD
    let bookinfo_openbd = fetchBookSummary_openbd(isbn);
    if(bookinfo_openbd){
      let bookinfo_openbd_sm = bookinfo_openbd.summary;
      var now = getNowDate();
      let bookinfo_openbd_sb=bookinfo_openbd.onix.DescriptiveDetail.TitleDetail.TitleElement["PartNumber"];
      if(bookinfo_openbd_sb){
        bookinfo_openbd_sm.title = bookinfo_openbd_sm.title + " " + bookinfo_openbd_sb;
      }
      if(bookinfo_openbd_sm && bookinfo_openbd_sm.author){
        if(bookinfo_openbd_sm.cover){
          sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_openbd_sm.title, bookinfo_openbd_sm.publisher, getDate(bookinfo_openbd_sm.pubdate), bookinfo_openbd_sm.cover, bookinfo_openbd_sm.author, now]]);
        }else{
          //画像なかったらGoogleの画像を取得
          let bookinfo_google=fetchBookSummary_google(isbn);
          if(bookinfo_google["volumeInfo"]["imageLinks"]){
            sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_openbd_sm.title, bookinfo_openbd_sm.publisher, getDate(bookinfo_openbd_sm.pubdate), bookinfo_google["volumeInfo"]["imageLinks"].thumbnail, bookinfo_openbd_sm.author, now]]);
          }else{
            //Googleにも画像がない
            sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_openbd_sm.title, bookinfo_openbd_sm.publisher, getDate(bookinfo_openbd_sm.pubdate), "", bookinfo_openbd_sm.author, now]]);
          }
        }
      }
    }else{
      //楽天
      let bookinfo_rakuten = fetchBookSummary_rakuten(isbn);
      if(bookinfo_rakuten){
        bookinfo_rakuten = bookinfo_rakuten["Item"];
        let bookinfo_google=fetchBookSummary_google(isbn);
        if(bookinfo_rakuten.largeImageUrl){
          sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_rakuten.title, bookinfo_rakuten.publisherName, bookinfo_google["volumeInfo"].publishedDate, bookinfo_rakuten.largeImageUrl, bookinfo_rakuten.author, now]]);
        }else{
          //画像なかったらGoogleの画像を取得
          let bookinfo_google=fetchBookSummary_google(isbn);
          if(bookinfo_google["volumeInfo"]["imageLinks"]){
          sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_rakuten.title, bookinfo_rakuten.publisherName, bookinfo_google["volumeInfo"].publishedDate, bookinfo_google["volumeInfo"]["imageLinks"].thumbnail, bookinfo_rakuten.author, now]]);
          }else{
            //Googleにも画像がない
            sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_rakuten.title, bookinfo_rakuten.publisherName, bookinfo_google["volumeInfo"].publishedDate, "", bookinfo_rakuten.author, now]]);
          }
        }
      }else{
        //Google
        let bookinfo_google=fetchBookSummary_google(isbn);
        if(bookinfo_google){
          if(bookinfo_google["volumeInfo"]["imageLinks"]){
            sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_google["volumeInfo"].title, (bookinfo_google["volumeInfo"].publisher)?bookinfo_google["volumeInfo"].publisher:"", bookinfo_google["volumeInfo"].publishedDate, bookinfo_google["volumeInfo"]["imageLinks"].thumbnail,  bookinfo_google["volumeInfo"]["authors"].join(","), now]]);
          }else{
            //Googleにも画像がない
            sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, bookinfo_google["volumeInfo"].title, (bookinfo_google["volumeInfo"].publisher)?bookinfo_google["volumeInfo"].publisher:"", bookinfo_google["volumeInfo"].publishedDate, "",  bookinfo_google["volumeInfo"]["authors"].join(","), now]]);
          }
        }else{
          //どこもヒットしない
          sheet.getRange(i + 1, 1, 1, row.length).setValues([[i, isbn, "", "", "", "", "", now]]);
        }
      }
    }
  });
}
```

# AppSheetについて
個人的にですが、IDの所は[元記事](https://qiita.com/takatama/items/fa141cb13ccc7324f887)では **UNIQUEID()** を指定していましたが、スプレットシート上でもわかりやすく見れるように **[_RowNumber]** に変更しています。
※それに伴い、TypeもNumberに変更しています。
ISBNの重複を避けるためにISBNのKEYにチェックを入れています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/855481/c0292d74-2f96-67c1-25d8-469fa4c9db41.png)

# 最後に
あとは[元記事](https://qiita.com/takatama/items/fa141cb13ccc7324f887)と同じように動作検証していけば完了となります。

[元記事](https://qiita.com/takatama/items/fa141cb13ccc7324f887)をまねていざ作ってみたはいいもののOpen BDでは意外とヒットしない
ことがあったため他のAPIも使えばいいんじゃないかという考えのもと作成してみました。
GASやJSONの勉強にもなり面白かったです。

# 追記(2022/4/25)
すべてにおいて画像がない場合は、
Amazonの商品ページに載っている画像のアドレスを、
スプレッドシートにコピペすることで、
アプリ側にも画像が表示されるようになります。
※AmazonのAPIは色々と制約があるようなので、
 今回は採用見送っています。