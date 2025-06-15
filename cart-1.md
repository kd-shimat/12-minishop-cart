---
sort: 2
---

# 仕様書③ : カート内の商品画面

 **「仕様書①,②」で作成した`public`ディレクトリ内のソースコードを、今回colneした`public`ディレクトリにコピーしてください。**
「仕様書①,②」で作成したソースコードをコピーすると、以下のようなディレクトリ構成となります。

```text
public
├── classes
│   ├── dbdata.php
│   └── product.php
├── css
│   └── minishop.css
├── images(中のファイル名は省略)
├── index.php
└── product
    ├── product_detail.php
    └── product_select.php
```

以降からは、このディレクトリ構成を前提として作業を進めていきます。

## カートの概要説明

カート内の商品画面(`cart_list.php`)には、以下の３つの機能が順番に追加されます。

- カート内に商品を追加する・・・仕様書③で実装する←本章はここ
- カート内の商品を表示する・・・仕様書③で実装する←本章はここ
- 特定の商品をカートから削除する・・・仕様書④で実装する
- カート内の商品の注文数を変更する・・・仕様書④で実装する

![](./images/cart_transition.png)

## 本章から追加されたテーブルについて

前章で作成したテーブルitemsに加え、新たにテーブルcartを作成します。

**テーブル名：cart**

| カラム名 | データ型 | 制約 | 備考 |
| - | - | - | - |
|ident|int型|主キー|商品番号|
|quantity|int型||注文数|

## カート内の商品画面の作成

カート内に商品が登録され、画面に表示するまでの一連の簡単な流れを示します。

### ◆商品詳細画面

- 「カートに入れる」ボタンをクリック
- 商品番号と注文数を送信

![](./images/product_detail_display.png)

### ◆カート内の商品画面

- 商品番号と注文数を受け取り、テーブルcartに登録
- テーブルcart内のすべてのデータを抽出し、画面に表示

**単一の商品がカート内に入っている場合**
![](./images/cart_list_display_book.png)

**複数の商品がカート内に入っている場合**
![](./images/cart_list_display_three.png)

上記の処理の流れですが、よりプログラム的に表現すると以下のようになります。<br>
※実際の開発現場でも、抽象的なものをよりプログラム的な表現に変換することが求められますので、脳内変換しながら練習してみてください。

1. `classes/cart.php` にクラス`Cart`を宣言し、以下の2つのメソッドを定義
   1. `addItem`メソッド
      - 商品番号をキーにテーブルitemsから商品データを抽出
      - 注文数とあわせてテーブルcartに登録

   2. `getItems`メソッド
      - テーブルcart内のすべてのデータを抽出

2. 商品詳細画面(`product_detail.php`)から送られてくるリクエストを受け取る、商品追加(`cart_add.php`)を作成
   1. 商品詳細画面(`product_detail.php`)から送られてきた商品番号と注文数を取得
   2. クラス`Cart`のオブジェクトを生成
   3. `addItem`メソッドを呼び出し、テーブルcartに商品データと注文数を登録
   4. カート内の商品を表示する「`cart_list.php`」を読み込む

3. カート内の商品画面(`cart_list.php`)に、以下の処理を記述
   1. Cartオブジェクトの`getItems`メソッドを呼び出し、テーブルcart内のすべてのデータを抽出
   2. 抽出したテーブルcart内のすべてのデータを画面に表示(その際、金額と合計金額も計算し表示すること)

## カート内に関するデータベース操作を行うクラスCart

データベースの基本事項を定義するクラス`DbData`を継承し、カートに関するデータベース操作を行うクラス`Cart`を定義します。
定義するファイルは、`classes/cart.php` で、このクラス`Cart`には以下の２つのメソッドを定義します。

1. 商品番号をキーにテーブルitemsから商品データを抽出し、注文数とともにテーブルcartに登録するメソッド

```text
アクセス修飾子： public
メソッド名： addItem
引数： $ident(商品番号)、$quantity(注文数)
戻り値： なし
```

2. テーブルcart内のすべてのデータを抽出するメソッド

```text
アクセス修飾子： public
メソッド名： getItems
引数： なし
戻り値： カート内のすべてのデータの結果セット
```

上記の仕様から、クラス`Cart`を定義する「`cart.php`」を作成します。

**classes/cart.php**

```php
<?php
// スーパークラスであるDbDataを利用するため
require_once __DIR__ . '/dbdata.php';

class Cart extends DbData
{
  // 商品をカートに入れるメソッド
  // 【ヒント】商品番号($ident)と注文数($quantity)をテーブルcartに登録する処理①、②を記述する
  public function addItem($ident, $quantity)
  {
    // ① SQL文を定義する
    // ② 実行する
  }

  // カート内のすべてのデータを取り出すメソッド
  // 【ヒント】テーブルcart内のすべてのデータを抽出し、商品テーブルitemsの結果セットを返す処理①〜④を記述する
  public function getItems()
  {
    // ① SQL文を定義する
    // ② 実行する
    // ③ 結果セットを取り出す
    // ④ 結果セットを戻り値として返す
  }
}
```

【ヒント】 `getItems`メソッドで定義するSQL文は以下のとおりです。
テーブルcartの商品番号から商品テーブルitemsの項目を取り出す必要があるため、以下のようにJOINを使い、テーブルを結合し取得します。

```php
$sql = "SELECT items.ident, items.name, items.maker, items.price, cart.quantity, items.image, items.genre FROM cart JOIN items ON cart.ident = items.ident";
```

## 商品追加機能(cart_add.php)

※ファイルを作成する前に、1つ注意事項があります。
今後作成する **cart_xxx.php** というファイル名のファイルは、すべて`cart`ディレクトリに配置してください。
また、`cart`ディレクトリは、`public`ディレクトリに配置してください。

商品追加機能(`cart_add.php`)には、以下の処理を記述します。

1. 商品詳細画面(`product_detail.php`)から送られてきた商品番号と注文数を受け取る
2. クラス`Cart`のオブジェクトを生成する
3. Cartオブジェクトの`addItem`メソッドを呼び出し、テーブルcartに商品番号と注文数を登録する
4. カート内の商品を表示するカート内の商品画面(`cart_list.php`)を読み込む

**cart/cart_add.php**

```php
<?php
    // 送られてきた商品番号と数量を受け取る
    $ident = $_POST['ident'];
    $quantity = $_POST['quantity'];

    // Cartオブジェクトを生成し、カートに入れるメソッドを呼び出す
    require_once  __DIR__  .  '/../classes/cart.php';
    $cart = new Cart( );
    $cart->addItem($ident, $quantity);

    // cart_list.phpを読み込み、カート内の商品を画面に表示する
    require_once __DIR__ . '/cart_list.php';
```

## カート内の商品画面(cart_list.php)

この画面の完成形は以下のとおりです。(画面は3つの商品がカートに入っている場合です。)

![](./images/cart_list_display_three.png)

この画面を参考に、カート内の商品画面(`cart_list.php`)を作成します。
処理概要についてはコメントで記載していますので、そちらを参考に記述してください。

**cart/cart_list.php**

```php
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>ショッピングサイト</title>
    <link rel="stylesheet" href="../css/minishop.css">
</head>
<body>
<h3>カートの商品</h3>
<table>
  <tr>
    <th>&nbsp;</th>
    <th>商品名</th>
    <th>メーカー・著者<br>アーティスト</th>
    <th>価格</th>
    <th>注文数</th>
    <th>金額</th>
  </tr>

  <!-- ====ここから以下の2つの処理を記載してください==== -->
  <!-- 1. CartオブジェクトのgetItemsメソッドを呼び出し、テーブルcart内のすべてのデータを抽出する。 -->
  <!-- ※なお、cart_add.phpで既にCartオブジェクトを生成しているため、Cartオブジェクトの再生成は不要。 -->



  <!-- 2. 抽出したテーブルcart内のすべてのデータを画面に表示する。 -->
  <!-- 商品一覧の表示に関しては、ジャンル別商品一覧画面(product_select.php)を -->
  <!-- 参考にすると作りやすい。ただし、注文数欄、金額欄、合計金額欄の追加を忘れないこと。 -->



  <!-- =========================================== -->

<br>
<a href="../index.php">ジャンル選択に戻る</a>&nbsp;&nbsp;<a href="../order/order_now.php">注文する</a>

</body>
</html>
```

【ヒント】画面に表示するにあたり、カート内の商品一覧表のセル(`<td>`や`<th>`)には以下の`class`属性の設定を以下のように行います。
なお、[「仕様書① : ジャンル選択画面、ジャンル別商品一覧画面」の**`<td>`セル内のクラス設定について**](../ec-site-i/README.md####tdセル内のクラス設定について)で説明した内容と同様のものです。

- 画像: `<td class="td_mini_img">` また、`<img>` には `class="mini_img"` を設定する。
- 商品名: `<td class="td_item_name">`
- メーカー・著者・アーティスト: `<td class="td_item_maker">`
- 価格: `<td class="td_right">`
- 注文数: `<td class="td_right">` ←本章で追加
- 金額: `<td class="td_right">` ←本章で追加
- 合計金額: `<th colspan="5">` ←本章で追加

## 動作確認(1回目)

商品詳細画面(`product_detail.php`)の「カートに入れる」ボタンをクリックすると、その商品がテーブル cart に登録され、カート内の商品画面(`cart_list.php`)が表示されることを確認してください。

**但し、この時点ではある「バグ」が潜んでいます。**
この動作確認が完了しましたら、バグの修正にうつります。

商品詳細画面：product_detail.php

![](./images/product_detail_display.png)

カート内の商品画面：cart_list.php

![](./images/cart_list_display_book.png)

複数の商品が入っている場合

![](./images/cart_list_display_three.png)