# Playwright スクレイピング 頻出機能まとめ



```python
from playwright.sync_api import sync_playwright

# ブラウザ・ページの起動
with sync_playwright() as pw:
    browser = pw.chromium.launch(headless=False)
    context = browser.new_context(
        viewport={'width': 1920, 'height': 1080},
    )
    page = context.new_page()

# 基本遷移
page.goto('https://example.com')

# DOM構築完了まで待つ（JS不要な静的サイトに最適）
page.goto('https://example.com', wait_until='domcontentloaded')

# 現在のURL
page.url

# 戻る・進む
page.go_back()
page.go_forward()

# 単一要素（ElementHandle）
elem = page.query_selector('h1')

# 複数要素
elems = page.query_selector_all('ul > li')

# 要素起点で絞り込む
child = elem.query_selector('.child')

# 存在チェック
if page.query_selector('.banner'):
    ...

# textContent（非表示テキスト含む、改行・空白そのまま）
text = elem.evaluate('el => el.textContent')

# innerText（表示されているテキストのみ、ある程度整形される）
text = elem.evaluate('el => el.innerText')

# 属性値
href = elem.get_attribute('href')
src  = elem.get_attribute('src')
cls  = elem.get_attribute('class')


# セレクタが出現するまで待機
# タイムアウト指定（ミリ秒）
page.wait_for_selector('.result', timeout=10000)

# クリック
elem.click()

# テキスト入力（既存テキストを消去してから入力）
elem.fill('検索ワード')

# キー操作
page.keyboard.press('Enter')
page.keyboard.press('Tab')


# リソース種別でブロック（高速化）
def handler(route):
    if route.request.resource_type in {'image', 'font'}:
        route.abort()
    else:
        route.continue_()

page.route('**/*', handler)

# 特定URLパターンをブロック
page.route('**/*.{png,jpg,gif}', lambda r: r.abort())


# ページ全体のHTML（JS実行後のDOM）
html = page.content()

# 特定要素のinnerHTML
inner = elem.inner_html()

# ページ上でJSを実行
title = page.evaluate('document.title')

# 要素に対してJSを実行
text = elem.evaluate('el => el.textContent.trim()')

# 複雑なJS
data = page.evaluate('''() => {
    return Array.from(document.querySelectorAll('a'))
        .map(a => a.href)
}''')


# ページ最下部までスクロール（無限スクロール対策）
page.evaluate('window.scrollTo(0, document.body.scrollHeight)')

# 要素をビューに入れる
elem.scroll_into_view_if_needed()

# iframe関連
frame = page.frame_locator('iframe#target')
frame = page.frame_locator('iframe#target').nth(0)
elem  = frame.locator('.content').element_handle()
elems  = frame.locator('.content').element_handles()
frame.locator(".content").wait_for(timeout=10000)


# 新しいタブが開くのを待ち受ける
# 「これから新しいタブが開くはずなので、開いたら捕まえておいて」という待ち受け宣言
with context.expect_page() as new_page_info:
    elem = page.query_selector('a[target="_blank"]')
    # 実際にリンクをクリックして新タブを発生させる
    elem.click()
# 捕まえておいた新タブをPageオブジェクトとして取り出す
new_page = new_page_info.value
# 新タブのページ読み込みが完了するまで待つ
new_page.wait_for_load_state()
# これ以降は new_page を普通の page と同じように操作できる
# pageとnew_pageはそれぞれ独立したPageオブジェクトなので、どちらに対してメソッドを呼ぶかで操作対象のタブが変わる。
# タブを3つ以上開いた場合も同様で、変数名が違うだけで全部同じように扱える


# イベントリスナーの登録
# 。「〇〇が起きたらこの関数を実行して」という仕組み
# page.on('イベント名', 実行する関数)

# ページ上でダイアログ（alert/confirm/prompt）が出たとき、自動でOKボタンを押す
page.on('dialog', lambda dialog: dialog.accept())

dialog.accept()   # OKボタン（confirmならYes、promptなら入力して確定）
dialog.dismiss()  # キャンセルボタン

# promptに文字列を入れて確定したい場合はこう書ける
page.on('dialog', lambda dialog: dialog.accept('入力テキスト'))

# ページ全体のスクショ
page.screenshot(path='page.png', full_page=True)

# 特定要素のスクショ
elem.screenshot(path='elem.png')
```


