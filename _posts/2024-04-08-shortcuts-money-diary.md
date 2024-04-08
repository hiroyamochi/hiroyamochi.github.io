---
title: iOSのショートカットから登録できる家計簿ツールつくった
date: 2024-04-08 15:00:00 +0900
categories: [Misc]
tags: [misc]
---
## はじめに
自分用の家計簿ツールをつくった．

求めるものは以下の2つ．
- とにかく簡単に入力できること
- Google スプレッドシートで管理できること

世の中にはいろんな家計簿ツールがあるけど，複数端末間でデータを扱えなかったり，一部の電子マネー等にしか対応してなかったりするのが手を出しづらいと感じているので，自分用につくってみようと思った．

## 今回使ったツール
- Google スプレッドシート
- Google Apps Script (GAS)
- iOS ショートカット

## ショートカット側の実装

おおまかな流れ
1. ショートカットを起動する
2. カテゴリ→金額→詳細(省略可) の順に入力
3. GASに対してHTTPのGETリクエストを送る

![alt](/assets/img/240408_shortcut1.png)

ショートカットアプリはあくまでもインタフェースとして使うだけ．

ホーム画面に置いたアイコンから起動するとすぐに入力プロンプトが出てくるので，ぽちぽち入れていく．

レシピは[こちら](https://www.icloud.com/shortcuts/d831adcb4121416eab0e9d6973f35d1b)から追加できるのでどうぞ．

大まかな使いかたはこちらにも載せてます．

https://github.com/hiroyamochi/money-diary-with-shortcuts

## GAS側の実装

こっちはまったくの初心者なので間違いや非効率さは全然あるということをお断りしておく．

`doGet()` 関数を使うことでGETリクエストを処理できるらしい．

ショートカットから送られた入力を，スプレッドシートの `data` シートに日付とともに記載していく．

```js
function doGet(e) {
  var content = e.parameter.content;
  var amount = e.parameter.amount;
  var category = e.parameter.category;

  // スクリプトをくっつけたスプレッドシートを取得
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName("data");

  // 現在の最終行を取得
  var lastRow = sheet.getLastRow();

  // データを追加する行を選択
  var newRow = lastRow + 1;

  // 今日の日付を取得
  var today = new Date();

  // データを書き込む
  sheet.getRange("A" + newRow).setValue(today);
  sheet.getRange("B" + newRow).setValue(category);
  sheet.getRange("C" + newRow).setValue(amount);
  sheet.getRange("D" + newRow).setValue(content);
}

```

## スプレッドシート側の実装

`data` と `main` という2つのシートから成る．

`data` シートにはショートカットから送られたデータがGAS経由で書き込まれる．
![alt](/assets/img/240408_ss.png)

`main` シートは，`data` シートを参照して月ごと・カテゴリごとの金額を集計する．
![alt](/assets/img/240408_main.png)

集計にはこのような関数を使っている．

```
# 上記B2セルの場合

=sumifs(
data!$C:$C,
data!$B:$B, B$1, 
data!$A:$A, ">="&date(year($A2), month($A2), 1), 
data!$A:$A, "<="&eomonth(date(year($A2), month($A2), 1), 0)
)
```

自分がつくったシートは以下のリンクに置いておくので，使いたければどうぞ．

[自作スプレッドシート](https://docs.google.com/spreadsheets/d/1d7UOEItv7y8aqK0DoTNbIn-rak_Q76Fuysm4hnNh_oA/edit?usp=sharing)


## 展望
家計簿ツールのくせに収入の記録欄が無いという致命的なつくりかけなので，さっさとつくる．

デザイン面のセンスが終わってるので見やすいシートにもしたい．

ショートカット側のエラーハンドリングも最小限しか行っていないので，そちらも改善したい．
