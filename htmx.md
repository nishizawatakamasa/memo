# htmx覚書

* 目次
    * [概要](#概要)
    * [hx-*属性](#hx-*属性)
        * [hx-get系](#hx-get系)
        * [hx-trigger系](#hx-trigger系)
        * [hx-target系](#hx-target系)
        * [hx-swap系](#hx-swap系)


<a id="概要"></a>
## 概要

* hx-get、hx-trigger、hx-target、hx-swapのような独自の属性を通じてサーバーとやりとりする。  
* サーバ側でHTMLをレンダリングするため、テンプレート・エンジンとの相性が非常に良い  
* 実態はただのJavaScript。  
* ローカルに単一ファイルとして保存し、scriptタグで読み込むだけ。  

コンセプト
* aタグとformタグ以外も、HTTPリクエストを実行できてもいいはずだ。
* clickとsubmit以外のイベントが、HTTPリクエストのトリガーになってもいはずだ。
* GETとPOST以外のメソッドが使えてもいいはずだ。
* ページ全体を差し替えなくてもいいはずだ。

例：
```html
 <button
    hx-post="/clicked"
    hx-trigger="click"
    hx-target="#parent-div"
    hx-swap="outerHTML">
    Click Me!
</button>
 ```

<a id="hx-*属性"></a>
## hx-*属性

* htmxによって拡張された、「hx-」が頭に付く属性。
* htmxエンジンが直接解釈して処理する。
* HTMLのセマンティクスを損なわずに機能を拡張できる。
* HTMLのカスタムデータ属性のdata-*属性は付けるべきではない。


<a id="hx-get系"></a>
### hx-get系
AJAXリクエストを発行できるようにする属性

* 属性
    * hx-get
    * hx-post
    * hx-put
    * hx-patch
    * hx-delete

要素がトリガーされると、指定したURLに指定されたタイプのAJAXリクエストを発行する。


<a id="hx-trigger系"></a>
### hx-trigger系
hx-trigger

AJAXリクエストの発行をトリガーするものを指定。
属性を省略すると、デフォルトのイベントでリクエストを発行する。

デフォルトのイベント
|||
|-|-|
|form|submit|
|input<br>select<br>textarea|change|
|それ以外|click|


トリガー値は次のいずれか。

1.イベント名
#### 1-1 標準イベント
click、mouseover、keydown、load等。

#### 1-2 標準イベントフィルター
標準イベント名の後に、角括弧で囲んだJavaScriptの式を指定できる。
このJavaScriptの式は真偽値を返し、trueの場合のみイベントがトリガーされる。
```html
<div hx-get="/clicked" hx-trigger="click[ctrlKey]">Control Click Me</div>
```

#### 1-3 標準イベント修飾子
標準イベントには、動作を変更する修飾子も設定できる。
修飾子の例
```html
<!-- イベントは1回しかトリガーされない。 -->
<div hx-trigger="click once"></div>
<!-- イベントがリクエストをトリガーする前に遅延が発生する。 -->
<div hx-trigger="load delay:5s"></div>
```

#### 1-4 非標準イベント
htmx独自のイベント。
例：
* revealed(要素がviewport内に入ったら)

2.カンマ区切りの複数イベント  
カンマで区切って複数のイベントを指定できる。    
各トリガーには独自のオプションを付けられる。    

例：
```html
<!-- ページの読み込み時にすぐに読み込まれ、クリックするたびに1秒遅れて再度読み込まれる。 -->
<div hx-get="/news" hx-trigger="load, click delay:1s"></div>
```

3.特殊な指定  
hx-trigger="every 2s"で「2秒間隔でリクエストを実行」となる。  
秒数は任意の値を指定。一定間隔でのデータ取得が必要な場合に便利。  
例：
```html
<div hx-get="/latest_updates" hx-trigger="every 2s [someConditional]">
  Nothing Yet!
</div>
```


<a id="hx-target系"></a>
### hx-target系

サーバからのレスポンス（HTML）をどの要素に差し込むかを指定する。  
指定する値はCSSセレクタ。  
省略した場合、「自分自身」がターゲットになる。  



<a id="hx-swap系"></a>
### hx-swap系

サーバからのレスポンス（HTML）を要素に差し込む方法を指定する。  
以下が指定できる値。デフォルトはinnerHTML。
* innerHTML：対象要素の内部HTMLと交換
* outerHTML：ターゲット要素全体と交換
* beforebegin：ターゲット要素の前に追加
* beforeend：ターゲット要素の最後の子の後に追加
* afterbegin：ターゲット要素の最初の子要素の前に追加
* afterend：ターゲット要素の後に追加
* textContent：ターゲット要素のテキストコンテンツと交換
* delete：ターゲット要素を削除。レスポンスは考慮されない。
* none：差し替えをしない

修飾子
```html
<!-- 新しいコンテンツで見つかったタイトルは無視され、ドキュメントのタイトルは更新されない。 -->
 <button hx-post="/like" hx-swap="outerHTML ignoreTitle:true">Like</button>
<!-- 応答を受信して​​からコンテンツをスワップするための htmx の待機時間を変更 -->
<div hx-get="/example" hx-swap="innerHTML swap:1s">Get Some HTML</div>
<!-- スワップとセトルのロジック間の時間を変更できます -->
<div hx-get="/example" hx-swap="innerHTML settle:1s">Get Some HTML</div>
<!-- 控えめなスクロールで要素を表示。表示要素はセレクタ等で指定(省略するとターゲット要素)。topかbottomを指定。 -->
<div hx-get="/example" hx-swap="innerHTML scroll:#another-div:bottom">Get Some HTML</div>
<!-- 要素を必ず表示。表示要素はセレクタ等で指定(省略するとターゲット要素)。topかbottomを指定。 -->
<div hx-get="/example" hx-swap="innerHTML show:window:top">Get Some HTML</div>
<!-- ターゲット内のフォーカスされた要素が、自動的にビューポートにスクロールインするようになる。 -->
<input id="name" hx-get="/validation" hx-swap="outerHTML focus-scroll:true"/>
```
