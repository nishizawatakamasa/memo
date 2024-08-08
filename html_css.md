# HTML/CSS覚書

## HTMLの原則
タグにはそれぞれ役割があり、適当に割り振ってはいけない。  
hタグであれば、見出しの役割があり、pタグであれば段落の意味がある。  
Webサイトを制作する上では、HTMLタグの役割と意味を理解して正確にコーディングしなければならない。  
idは機能用、classは装飾用につける。  
JavaScript用とCSS用のクラスを区別するのも有用。  
例：
```html
<div class="example js_example"></div>
```

## CSSの原則
classに対してスタイルを当てるのが基本。  
idにスタイルは当てない。  
タグにスタイルを当てる場合は十分にスコープを絞り込む。  
!importantは使用しない。  

## HTMLの頻出タグ
```html
<!-- 構造 -->
<header></header>
<main></main>
<footer></footer>
<nav></nav>
<aside></aside>
<h1></h1>
<h2></h2>
<h3></h3>
<h4></h4>
<h5></h5>
<h6></h6>
<section></section>
<article></article>

<!-- グループ化 -->
<div></div>
<span></span>

<!-- 文章 -->
<p></p>
<br>

<!-- 埋め込み -->
<img>
<iframe></iframe>

<!-- リンク -->
<a></a>

<!-- 並べる -->
<ul></ul>
<ol></ol>
<li></li>
<table></table>
<thead></thead>
<tbody></tbody>
<tfoot></tfoot>
<tr></tr>
<th></th>
<td></td>

<!-- 入力フォーム -->
<form></form>
<input>
<textarea></textarea>
<select></select>
<option></option>
```

## CSSの基本単位
|単位|長さ|
|:-|:-|
|px|1ピクセル|
|em|その要素のfont-size|
|rem|ルート要素のfont-size(room emの略)|
|vh|ビューポートの高さの1%(Viewport Heightの略)|
|vw|ビューポートの幅の1%(Viewport Widthの略)|

Q. ビューポートとは？  
A. Webブラウザの表示領域のこと


## 擬似クラスと疑似要素
擬似クラス→要素の状態。  
疑似要素→擬似的に追加された要素。  
これらは完全に別の意味。  

* ::before, ::afterを指定する場合の注意点
    1. テキストや記号を擬似要素として挿入したい場合はcontentを指定し、その中に記述(content: "あいうえお";)。
    1. テキストや記号以外の要素や画像を挿入したい場合はcontentを空(content: "";)にし、擬似要素を付けた要素にposition: relative;、擬似要素にposition: absolute;を指定。

## 要素(=タグ)の構成

### 基本
* font-size (文字の大きさ)
    * %: 親要素のフォントサイズ基準
* color (文字色)
* background (背景)
* contents (コンテンツ)
* border (枠線)
    * border-width:基本px指定。%指定不可。

#### backgroundの範囲ってどこまで？ → background-clipプロパティの値で決まる
|background-clipの値|backgroundの範囲|
|:-|:-|
|border-box|**初期値** ボーダーの外側の辺まで。(ボーダーが手前に来る)。|
|padding-box|パディングの外側の辺まで。|
|content-box|コンテンツの外側の辺まで。|

### 余白、幅、高さの単位
|プロパティ|%|auto|
|:-|:-|:-|
|padding|親要素(グリッドエリア)の幅基準|指定不可|
|左右のmargin|親要素(グリッドエリア)の幅基準|使用可能なスペースのすべて|
|上下のmargin|親要素(グリッドエリア)の幅基準|0(親がflexかGridの場合は使用可能なスペースのすべて)|
|width|親要素(グリッドエリア)の幅基準|**初期値** 親要素の幅いっぱい(display: inline-block;の時は子要素のコンテンツ分)|
|height|親要素(グリッドエリア)の高さ基準|**初期値** 子要素のコンテンツ分|

※width: fit-content;で子要素のコンテンツ分

#### widthとheightってどこの幅と高さ？ → box-sizingプロパティの値で決まる
|box-sizingの値|widthとheightの範囲|
|:-|:-|
|border-box|ボーダーの外側の辺まで。|
|content-box|**初期値** コンテンツの外側の辺まで。|

### displayプロパティごとの挙動の違い
||並び方|幅、高さ指定|余白指定|
|:-|:-|:-|:-|
|display: block;|縦|可|可|
|display: inline-block;|橫|可|可|
|display: inline;|橫|不可|左右のみ可|

#### 初期値がdisplay: inline;の要素の例
* a
* span
* label
* br
* img※
* iframe※
* video※

※img,iframe,videoのdisplay初期値はinlineだが、置換要素なので例外的にinline-blockの挙動になる。
#### 置換要素とは
別の場所にあるCSSから独立したコンテンツを、HTMLタグのコンテンツとして置き換えられるような要素のこと。

## レイアウト小技

### text-align
* テキスト中央寄せ
### float
* 要素を浮かせて回り込みを表現

## display: flex;
**横並びにする**

### 上下左右に寄せるプロパティ
* justify-content
    * start
    * center
    * end
    * space-between
    * space-around
    * space-evenly
* align-items(align-selfの同時指定)
    * start
    * center
    * end
    * baseline
* align-content ※wrap時
    * start
    * center
    * end
    * space-between
    * space-around
    * space-evenly

### 折り返し
* flex-wrap

## display: grid;
**格子状に分割する**

### 上下左右に寄せるプロパティ
* [justify|align]-[content|items|self]
    * start
    * center
    * end

### 領域指定
* grid-template: ;
* grid-template-columns: ;
* grid-template-rows: ;
### 配置指定
* grid-area: 2/2/4/5;
* grid-area: "area_name";

## position
**好きなところに配置するプロパティ**

### 値
* relative, absolute
    * 親要素にrelative子要素にabsoluteという関係でセットで使う。
    * absolute要素を、親要素のrelativeの範囲の中に固定して配置。
* fixed
    * パソコンやスマホなどの画面の中に固定して配置。スクロールしても付いてくる。
* sticky
    * 自動的に親要素がスティッキーコンテナとして定義される。
    * スクロール時に、指定した位置に固定して配置。
    * top指定必須。


### 固定位置の指定
* top, bottom
    * %指定の場合: 包含ブロックの高さ基準。
* left, right
    * %指定の場合: 包含ブロックの幅基準。
