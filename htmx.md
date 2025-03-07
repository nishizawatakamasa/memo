# htmx
## 概要



hx-get、hx-trigger、hx-target、hx-swapのような独自の属性を通じてサーバーとやりとりする。




HTMLの組み立てを原則サーバ側で行う
そのため、所謂HTMLのテンプレート・エンジンと相性が非常に良い



* コンセプト
    * aタグとformタグ以外も、HTTPリクエストを実行できてもいいはずだ。
    * clickとsubmit以外のイベントが、HTTPリクエストのトリガーになってもいはずだ。
    * GETとPOST以外のメソッドが使えてもいいはずだ。
    * ページ全体を差し替えなくてもいいはずだ。




レスポンスも、HTMLを返すのが前提となります。


実態はただのJavaScript。
ローカルにコピペして単一のファイルとして保存し、scriptタグで読み込むだけ。



```html
<!-- 例 -->
<script src="js/htmx.min.js" defer></script>
```



```html
 <button
    hx-post="/clicked"
    hx-trigger="click"
    hx-target="#parent-div"
    hx-swap="outerHTML">
    Click Me!
</button>
 ```


## hx-*属性
htmxによって拡張された、「hx-」が頭に付く属性。
htmxエンジンが直接解釈して処理する。
HTMLのセマンティクスを損なわずに機能を拡張できる。
HTMLのカスタムデータ属性のdata-*属性は付けるべきではない。


### hx-get系
AJAXリクエストを発行できるようにする属性
* 属性
    * hx-get
    * hx-post
    * hx-put
    * hx-patch
    * hx-delete

hx-post="/clicked"のように、AJAXリクエストを発行するURLを指定する.
要素がトリガーされると、指定されたURLに指定されたタイプのリクエストを発行する。



HTTPリクエストを実行するイベントを指定する属性


### hx-trigger系
hx-trigger

hx-triggerを省略すると、デフォルトのイベントで発火します。
formならsubmitイベント、input、select、textareaならchangeイベント、以外ならclickイベントがデフォルトです。


このhx-trigger属性を使用すると、AJAX リクエストをトリガーするものを指定できます。トリガー値は次のいずれかになります。

イベント名（例：“click” または “my-custom-event”）の後にイベントフィルタとイベント修飾子のセットが続きます。
フォームの投票定義every <timing declaration>
カンマ区切りのイベントリスト





標準イベント
click、mouseover、keydown



標準イベントフィルター
イベント名の後に、角括弧で囲んだJavaScriptの式を指定できる。
このJavaScriptの式は真偽値を返し、trueの場合のみイベントがトリガーされる。


<div hx-get="/clicked" hx-trigger="click[ctrlKey]">Control Click Me</div>



標準イベント修飾子
標準イベントには、動作を変更する修飾子も設定できる。
修飾子の例
```html
<!-- イベントは1回しかトリガーされない。 -->
<div hx-trigger="click once"></div>
<!-- イベントがリクエストをトリガーする前に遅延が発生する。 -->
<div hx-trigger="load delay:5s"></div>
```


非標準イベント
htmx独自のイベント。
load(要素が読み込まれたら)
revealed(要素がviewport内に入ったら)


複数のトリガー
複数のトリガーをコンマで区切って指定できます。各トリガーには独自のオプションがあります。

  <div hx-get="/news" hx-trigger="load, click delay:1s"></div>
この例では、/newsページの読み込み時にすぐに読み込まれ、クリックするたびに 1 秒遅れて再度読み込まれます。

イベントはカンマ区切りで複数指定することもできます。



特殊な指定として、hx-trigger="every 2s"があります。これで「2秒間隔でリクエストを実行」となります。秒数は任意の値を指定できます。一定間隔でデータ取得が必要な場合は便利ですね。
GETこの例では、URLに を1 秒ごとに発行し/latest_updates、その結果をこの div の innerHTML にスワップします。
ポーリングにフィルターを追加する場合は、ポーリング宣言の後に追加する必要があります。
<div hx-get="/latest_updates" hx-trigger="every 1s [someConditional]">
  Nothing Yet!
</div>






### hx-target系


hx-targetで、サーバからのレスポンス（HTML）を差し込む要素を指定します。指定する値は、CSSセレクタです。
省略した場合、「自分自身」がターゲットになります。





### hx-swap系

スワッピング
htmx では、DOM に返される HTML をスワップする方法がいくつか用意されています。デフォルトでは、コンテンツはターゲット要素の を置き換えます 。hx -swap属性を次のいずれかの値でinnerHTML使用して、これを変更できます。




・hx-swapについて
サーバからのレスポンス（html）をどこに差し込むかはhx-targetで指定しました。hx-swapで、「どのように差し込むか」を指定します。




innerHTML、outerHTMLはレスポンスと「交換」しますが、afterbegin、beforeendは「追加」されます。

noneは、レスポンスで差し替えをしない時に指定します。ページの表示に影響しないリクエストも当然あると思うので、案外使う頻度は高そうですね。

hx-swapを指定しない場合、innerHTMLがデフォルトになります。











innerHTML- 対象要素の内部HTMLを置き換える
outerHTML- ターゲット要素全体をレスポンスに置き換える
textContent- レスポンスをHTMLとして解析せずに、ターゲット要素のテキストコンテンツを置き換えます。
beforebegin- ターゲット要素の前にレスポンスを挿入する
afterbegin- ターゲット要素の最初の子要素の前にレスポンスを挿入します
beforeend- ターゲット要素の最後の子の後にレスポンスを挿入します
afterend- ターゲット要素の後にレスポンスを挿入します
delete- 応答に関係なくターゲット要素を削除します
none- 応答からコンテンツを追加しません (帯域外の項目は引き続き処理されます)。


修飾子
属性hx-swap
属性hx-swapは、スワップの動作を変更するための修飾子をサポートします。以下に概要を示します。

```html
<!-- 新しいコンテンツで見つかったタイトルは無視され、ドキュメントのタイトルは更新されません。 -->
 <button hx-post="/like" hx-swap="outerHTML ignoreTitle:true">Like</button>
<!-- 応答を受信して​​からコンテンツをスワップするための htmx の待機時間を変更 -->
<div hx-get="/example" hx-swap="innerHTML swap:1s">Get Some HTML & Append It</div>
<!-- スワップとセトルのロジック間の時間を変更できます -->
<div hx-get="/example" hx-swap="innerHTML settle:1s">Get Some HTML & Append It</div>
```

scroll: リストの更新や、コンテンツの追加など、要素がある程度近くに表示されている可能性が高い場合に適しています。ユーザーエクスペリエンスを損なわないように、控えめにスクロールしたい場合に適しています。

show: 重要な要素（新しいコメント、エラーメッセージなど）を確実にユーザーに表示させたい場合に適しています。コンテンツの場所に関わらず、必ず表示させたい場合に適しています。
topとを受け取りますbottom。
デフォではターゲットが対象。セレクタ指定で上書き的な
hx-swap="beforeend scroll:bottom">
hx-swap="innerHTML show:#another-div:top">
hx-swap="innerHTML show:window:top">
