# quickquery 相談メモ（Sonnet）

---

## 質問 1

@quickquery この自作ライブラリを読み、コンセプトを逸脱しない範囲で、改善や機能追加案などを考えてみて。要は、今はない、「こういうのどう？」というアイデアが欲しい。いいのがあれば。コンセプトのアイデアでもいいし、なんでもOK

---

## 回答 1

コードを隅々まで読めました。アイデアを整理します。

---

## quickquery 改善・機能追加アイデア

コンセプト（crawl → scrape → extract → clean を短く書くための自分用パイプライン）を踏まえたうえで考えました。

---

### 1. `QuickParser.meta(name)` — meta_html との対称ショートカット

**今**:
```python
p.i('meta[name="quickquery:url_index"]').attr('content')
```

**こうなる**:
```python
p.meta('url_index')  # 'quickquery:' プレフィックスを自動補完
```

`meta_html(...)` で埋めて `QuickParser` で読むパターンが extract.py に何度も登場する。書く量が多い割に意味が薄い部分なので、対称 API として `meta(name)` を持たせると clean になる。

---

### 2. `QuickNode.url` / `.src` — live/offline の API 対称性

`QuickElement` には `.url`・`.src`（`urljoin` 済み）があるが、`QuickNode` には `.attr('href')` しかない。オフラインではベース URL が分からないので `urljoin` はできないが、`QuickParser(parser, base_url=...)` でベースURLを渡せる形にすれば揃う。

```python
p = quick_parser(parser, base_url='https://example.com')
p.i('a').url  # urljoin が効く
```

---

### 3. `QuickElement.itext` — 可視テキスト（`inner_text`）

現状の `.text` は `text_content()` で、非表示要素の文字列も含む・空白も荒い。Playwright の `inner_text()` は CSS で `display:none` の部分を除いて整形してくれる。

```python
elem.itext   # inner_text() ベース（表示されてる文字だけ）
elem.text    # text_content() ベース（現状のまま）
```

scan のマッチ精度が上がる場面がある。

---

### 4. `append_jsonl` / `write_jsonl` — 追記に向いた形式

CSV だと大量データのリアルタイム追記でヘッダ管理が面倒。JSONL（1 行 = 1 JSON）は追記が素直で、scrape.py の「1件処理したら即書き出す」パターンに向いている。

```python
append_jsonl(here('out/results.jsonl'), {'url': ..., '価格': ...})
```

クロール中に落ちても既処理分は残る。Parquet への変換は後から `pandas.read_json(..., lines=True)` で可能。

---

### 5. `counter(max=N)` — ページネーションの上限ガード

```python
for page_num in counter(max=500):
    # 500 ページ超えたら自動 StopIteration
```

現状は無限ループで `break` を自分で書く。上限を明示できるとバグ（break 忘れ）や想定外のサイト変更（最終ページが消えた等）への安全網になる。

---

### 6. `Checkpoint` — クロール中断・再開

スクレイピングは途中で落ちる。「済みURL はスキップ」を毎回自分で書くのは面倒。

```python
with Checkpoint(here('checkpoint.txt')) as cp:
    for url in urls:
        if cp.seen(url):
            continue
        # 処理
        cp.mark(url)
```

内部はテキストファイルに 1 行 1 URL を書くだけ。シンプルだが効果は大きい。

---

### 7. `QuickElementGroup.unique_urls` — 重複 URL 排除

`p.ii('a').urls` は重複あり。重複削除 + 順序保持の `unique_urls` が欲しい場面（crawl の URL 収集）はよくある。

```python
urls = p.ii('a').unique_urls   # dict.fromkeys() で実装、1 行
```

---

### 8. `QuickElement.tag` — タグ名

現状タグ名を取る手段がない。条件分岐やデバッグで地味に欲しい。

```python
elem.tag   # 'div', 'a', 'span' など
```

live 側は `evaluate('el => el.tagName.toLowerCase()')`, offline 側は `node.tag`。

---

### 9. `scan` の `extract(pattern)` — マッチして正規表現抽出を 1 ステップで

extract.py の典型パターン:
```python
dd_text(r'価格')  # → テキスト取得後、pandas側で正規表現抽出
```

scan レベルで「テキストを取りつつ正規表現のキャプチャグループを返す」オプションがあると extract → clean を一気にできる場面がある。ただしこれは clean との境界をどこに引くかの設計判断でもあるので、やりすぎになるかもしれない。

---

### 10. `QuickNode.html` / `QuickElement.html` — HTML 断片アクセス

内部 HTML の文字列が欲しい場面（保存・正規表現・デバッグ）は稀にある。

```python
node.html   # selectolax の node.html
elem.html   # Playwright の inner_html()
```

---

## コンセプトレベルのアイデア

**live/offline の API を意図的に揃える方針を明文化する**

現状 `QuickElement` と `QuickNode` の API 差（`.url`・`.src` の有無など）はドキュメントにも記載がなく、使う側が都度確認する必要がある。「live 側にあるものは offline 側にも必ず対応を作る」という設計ルールを自分ルールとして決めておくと、将来の拡張時にぶれない。

---

## 優先度でいうと

実装コストが低く効果が高いのは **1（`meta()`）** と **5（`counter(max=N)`）** と **7（`unique_urls`）**。

実務的に一番刺さりそうなのは **6（`Checkpoint`）**。長いクロールで毎回書くパターンを消せる。

**2（offline の `url`/`src`）** は live との対称性という設計コンセプトとして面白い。

---

## 質問 2

思想レベルの良いアイデアなどはある？なんでも

---

## 回答 2

コードを眺めながら考えた思想レベルの話をいくつか。

---

## 「テキストがセレクターである」という思想

`scan.m(r'価格').n('dd')` というパターンはちょっと面白い哲学を持っている。

CSS セレクター（`.price-value`、`#bukken-detail-price`）はサイトの**実装の詳細**であって、サイトリニューアルで簡単に変わる。一方で `r'価格'` というラベルテキストは**コンテンツそのもの**で、人間が読む限り変わりにくい。

つまり「CSSクラス名は壊れやすいセレクター、ラベルテキストは壊れにくいセレクター」という逆説がある。今の quickquery はすでにこれを実践しているが、思想として意識すると「できるだけ scan.m を使い、CSS クラスへの依存は最小限にする」という設計判断が生まれる。

**コンセプト名にするなら：「人間が読めるセレクター優先」**

---

## 「HTML ファイルが唯一の真実」という思想

`meta_html(...)` で URL・日時などを HTML 内に埋め込んでいるのは、実はかなり洗練された設計思想を持っている。

暗黙の原則：**HTML ファイルさえ残っていれば、あとは全部再構築できる**。

CSV も Parquet も「HTML から導出した view」に過ぎない。extract.py を書き直せばいつでも再生成できる。この思想を徹底すると、「スクレイピングの成果物は HTML アーカイブであって、CSV や Parquet は使い捨ての中間物」という方向性が見えてくる。

この思想を押し進めると、`meta_html` に埋めるべき情報をもっと増やす（HTTPステータス、リダイレクト元URL、クロールの設定値など）という判断が自然に出てくる。

---

## 「時間を軸に加える」という思想

今のパイプラインは「ある時点のスナップショットを撮る」もの。でも不動産価格は変わる、物件は消える。`saved_at` のメタデータはすでにあるのに、時間軸が活かされていない。

「同じ URL を定期的にスクレイピングして、差分を検出する」という発想を持ち込むと：

- `{hash(url)}-{timestamp}.html` で複数回の保存を共存させる
- extract 時に「前回との差分（価格変動、物件消滅）」を出力する

データが「点」から「線」になる。不動産のような継続変化するデータでは特に意味がある。

---

## 「Null は意味を持つ」という思想

現在の空ラッパー（`QuickElement(None)`）は「見つからなかった」という事実だけを伝える。でも「見つからなかった理由」は：

- そもそも要素が存在しない（サイト仕様）
- セレクターが間違っている（バグ）
- ページ読み込みが失敗した（ネット障害）
- タイムアウトした

...と全然違う。全部 `None` に潰れると、どれが起きたのかわからない。

今のログには reason が出るが、返り値には乗らない。「None だったから何もしない」で終わるか「None だったのに何かがおかしい」を区別する手段がない。`QuickElement(None, reason='timeout')` のように、失敗に文脈を持たせると何が起きているか分かりやすくなる。Rust の `Result`、Haskell の `Maybe` の精神に近い。

ただし、これはライブラリの API を複雑にするリスクもある。ログで十分とも言える。

---

## 「ステージ境界をファイルで刻む」という思想の明文化

crawl → scrape → extract → clean が**ファイルを介してつながっている**のは偶然ではなくて、壊れにくさのための設計だと思う。各ステージが失敗しても前のステージの成果物は残る。ステージを再実行すれば続きからやり直せる。

これは「関数型パイプライン」や「イベントソーシング」の思想に近い。今の README はこれを暗黙にやっているだけで、明文化されていない。「quickquery のパイプラインは各ステージがファイル artifact を生成し、次のステージはそれを消費する」という原則として意識しておくと、新しいスクレイピングプロジェクトを設計するときの指針になる。

---

おもしろいと思うのは最初の二つ。「人間が読めるセレクター」と「HTML が唯一の真実」は、今の設計の中にすでに暗黙に入っていて、ただ言葉になっていない思想。名前をつけると設計判断のときに使える軸になる気がする。
