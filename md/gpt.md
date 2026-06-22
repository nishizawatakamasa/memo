# quickquery 相談メモ（GPT）

## 質問 1

@quickquery この自作ライブラリを読み、コンセプトを逸脱しない範囲で、改善や機能追加案などを考えてみて。要は、今はない、「こういうのどう？」というアイデアが欲しい。いいのがあれば。コンセプトのアイデアでもいいし、なんでもOK

## 回答 1

quickquery を読んだ感じ、コンセプトは README の「自分用・非汎用」そのままで、**短い DOM 記法で crawl → scrape → extract を速く書く個人用ツールキット**だと思いました。なので、大きな spider フレームワーク化より、「今 README の例で毎回手書きしている定型を 1 個短くする」方向が合いそうです。

```139:145:C:\n\nishizawa\quickquery\README.md
    dt_scan = p.ii('dt').scan
    dd_text = lambda pattern: dt_scan.m(pattern).n('dd').text
    return {
        'url_index': p.i('meta[name="quickquery:url_index"]').attr('content'),
        'saved_at': p.i('meta[name="quickquery:saved_at"]').attr('content'),
        'request_url': p.i('meta[name="quickquery:request_url"]').attr('content'),
```

### 優先度高そうなアイデア

#### 1. ラベル→値抽出ヘルパ（dt/dd 系）

README の extract で、毎回これを書いています。

```python
dt_scan = p.ii('dt').scan
dd_text = lambda pattern: dt_scan.m(pattern).n('dd').text
```

不動産・求人・EC など、日本語ラベル DOM ではこれが定番です。コンセプトにかなり合うので、例えば:

- `p.kv('dt', 'dd').text(r'価格')`
- `p.kv('th', 'td').text(r'所在地')`

みたいな薄いヘルパがあると、extract がかなり短くなります。`scan.m(...).n('dd')` の思想を壊さず、むしろ強化する方向です。

#### 2. 属性 scan

今の `.scan` は text 専用です。でも crawl では href、scrape では alt/src を見る場面が多いです。

```python
p.ii('a').scan_attr('href').mm(r'/detail/')
p.ii('img').scan_attr('alt').m(r'外観').src
```

text scan と同じ NFKC + 正規表現の流儀で、属性版があると自然です。既存の `urls` / `srcs` / `attrs` とも相性がいいです。

#### 3. 静的 parser 側の URL 解決

Live 側は `.url` / `.src` がありますが、`QuickNode` にはありません。scrape 後 HTML を extract するとき、相対 URL をそのまま扱えないのは少しもったいないです。

- `quick_parser(parser, base_url=...)`
- または HTML 内の `quickquery:final_url` meta から base を読む helper

Live / Static の API 対称性が上がります。

#### 4. scrape snapshot helper

README の scrape では、毎回この定型があります。

```python
html = meta_html({...}) + page.content()
write_text(here('html') / f'{hash_name(page.url)}.html', html)
```

さらに failed.csv 追記もセットです。ここは quickquery らしい「定番パターンの 1 関数化」向きです。

- `p.save_html(here('html'), meta={...})`
- 失敗時は `append_csv(...)` まで含める

### やや小粒でも効くアイデア

#### 5. scan の候補順マッチ

scrape 例にこういうのがあります。

```python
img_li = img_li_scan.m(r'外観') or img_li_scan.m(r'^(?!.*間取).*')
```

`scan.pick([r'外観', r'^(?!.*間取).*'])` みたいな書き方があると、現場感は保ったまま短くなります。

#### 6. Live 操作の最小追加

今は `time.sleep(3)` 前提ですが、scrape では最低限これだけ欲しくなりがちです。

- `click()`
- `fill()`
- `wait_after((1, 2))`

全部入れる必要はなく、「よく使う 2〜3 個だけ」が quickquery っぽいです。

#### 7. resume / skip helper

`failed.csv` はあるのに、再開用の読み込みは毎回自前です。

- `resume_csv(here('csv/failed.csv'), key='url_index')`
- 処理済み URL を skip する小関数

長時間 scrape では地味に効きます。

#### 8. extract 結果の型を少しだけ安定化

`write_parquet` は docstring でも型混在に弱いと書いてあります。全部 str に落とすオプションだけでもあると、extract 後の clean が楽です。

- `write_parquet(..., coerce_str=True)`

### コンセプトから外れやすいもの

- Scrapy 的な汎用 crawler
- 設定ファイル駆動 extractor
- async 化
- 大規模 plugin 機構

「短い記法 + 個人パイプライン + 失敗に強いスクリプト」からは少し離れます。

### 個人的な優先順位

1. **ラベル→値抽出ヘルパ**
2. **属性 scan**
3. **静的 parser の URL 解決**
4. **scrape snapshot / resume helper**

このあたりは、README の crawl / scrape / extract をそのまま短くできるので、コンセプト逸脱が少なく効果が大きいと思います。

---

## 質問 2

思想レベルの良いアイデアなどはある？なんでも

## 回答 2

ある。思想レベルなら、僕は **「スクレイピングライブラリ」ではなく「調査・採取のための小さい言語」** として育てるのが良さそうだと思いました。

一番しっくり来る軸はこれです。

### 1. DOM を「探す」より「周辺から拾う」

quickquery は CSS セレクタを厳密に当てるより、`scan.m(...).n(...).text` で **ラベルを見つけて近くの値を取る** のが強いです。

なので思想としては、

> ページ構造を知り尽くさなくても、人間が見て「ここ」と分かる場所を短く取る

を前面に出すとよさそうです。

たとえば `価格`、`所在地`、`外観`、`スタッフコメント` みたいな人間向けラベルから値を拾う。これは不動産、求人、店舗情報、商品詳細、自治体ページなどにかなり合います。

### 2. 「正確なクローラ」ではなく「証拠付き採取」

個人用スクレイピングで本当に困るのは、あとから「この値どこから取ったんだっけ？」となることです。

quickquery の scrape → HTML保存 → extract の流れはすでに良いので、思想として **evidence-first scraping** に寄せるのはかなり良いです。

つまり、

> データだけでなく、取得時点の HTML・URL・スクショ・画像・メタ情報を一緒に残す

という考え方。これは大規模クローラより、個人の調査・分析用途にすごく合っています。

### 3. Live と Static を同じ手癖で扱う

今すでに `QuickPage` と `QuickParser` が似た API になっています。ここは思想としてもっと強いです。

> ブラウザ上で試した取り方を、そのまま保存 HTML の extract に持っていける

これはかなりいいコンセプトです。普通は Playwright と BeautifulSoup/selectolax で書き方が分かれますが、quickquery はそこを「同じ手癖」にできる。

これは README でももっと推せる価値だと思います。

### 4. 失敗しても止まらない、でも痕跡は残す

`goto` が `None` を返す、`process_map` が例外を握って `None` を返す、`failed.csv` に残す、という雰囲気があります。

これも思想にできます。

> 雑に長時間回しても全体は止めない。失敗はログと CSV に寄せる。

個人用バッチでは「完璧な例外設計」よりこっちの方が実用的です。  
ただし無音失敗ではなく、痕跡を残す。これが quickquery らしい。

### 5. Notebook の前段としての抽出器

`clean.ipynb` 例を見ると、quickquery は最終分析まで抱えるより、

> HTML から粗い構造化データを作り、あとは notebook / pandas に渡す

という立ち位置が自然です。

なので思想としては「分析ライブラリ」ではなく、**分析可能な状態まで持っていく採取器**。  
Parquet 出力があるのもこの方向に合っています。

### 6. サイトごとの使い捨てコードを、少しだけ短くする

ここは重要で、汎用化しすぎない方が良いと思います。

quickquery の美味しさは、

> サイト固有の泥臭いコードを書くことを前提に、その泥臭さだけを短くする

だと思います。

「設定を書けばどのサイトでも取れる」ではなく、「どうせ個別対応する。そのとき毎回うんざりする部分だけ消す」。  
この思想は `自分用・非汎用` と相性がいいです。

### 7. "Readable One-Off Script"

コンセプト名っぽく言うならこれが近いです。

> 一回限りのスクリプトを、あとで読める形で速く書く

スクレイピングコードはだいたい使い捨てですが、後で修正・再実行することも多い。  
だから quickquery は「堅牢なフレームワーク」より、「短く、読めて、再実行できる一回限りコード」を支えると良さそうです。

### まとめ

個人的に一番強いコンセプト案は、

> **人間がページを読む感覚に近い、証拠付きスクレイピング用ミニ言語**

です。

この方向なら、`scan`、`n/p/o`、HTML保存、meta、screenshot、Parquet が全部一本の思想でつながります。
