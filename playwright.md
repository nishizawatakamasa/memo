# Playwright スクレイピング 頻出機能まとめ

---

## 1. ブラウザ・ページの起動

```python
from playwright.async_api import async_playwright

async with async_playwright() as pw:
    browser = await pw.chromium.launch(headless=False)
    context = await browser.new_context(
        viewport={'width': 1920, 'height': 1080},
    )
    page = await context.new_page()
```

---

## 2. ナビゲーション

```python
# 基本遷移
await page.goto('https://example.com')

# DOM構築完了まで待つ（JS不要な静的サイトに最適）
await page.goto('https://example.com', wait_until='domcontentloaded')

# 現在のURL
page.url

# 戻る・進む
await page.go_back()
await page.go_forward()
```

---

## 3. 要素取得

```python
# 単一要素（ElementHandle）
elem = await page.query_selector('h1')

# 複数要素
elems = await page.query_selector_all('ul > li')

# 要素起点で絞り込む
child = await elem.query_selector('.child')

# 存在チェック
if await page.query_selector('.banner'):
    ...
```

---

## 4. テキスト・属性の取得

```python
# textContent（非表示テキスト含む、改行・空白そのまま）
text = await elem.evaluate('el => el.textContent')

# innerText（表示されているテキストのみ、ある程度整形される）
text = await elem.evaluate('el => el.innerText')

# 属性値
href = await elem.get_attribute('href')
src  = await elem.get_attribute('src')
cls  = await elem.get_attribute('class')
```

---

## 5. 待機

```python
# セレクタが出現するまで待機
await page.wait_for_selector('.result')

# タイムアウト指定（ミリ秒）
await page.wait_for_selector('.result', timeout=10000)

# 指定時間待機（固定スリープ）
await page.wait_for_timeout(2000)  # 2秒
```

---

## 6. クリック・入力

```python
# クリック
await page.click('button.submit')
await elem.click()

# テキスト入力（既存テキストを消去してから入力）
await page.fill('input[name="q"]', '検索ワード')

# キー操作
await page.keyboard.press('Enter')
await page.keyboard.press('Tab')
```

---

## 7. ネットワーク制御

```python
# リソース種別でブロック（高速化）
async def handler(route):
    if route.request.resource_type in {'image', 'font'}:
        await route.abort()
    else:
        await route.continue_()

await page.route('**/*', handler)

# 特定URLパターンをブロック
await page.route('**/*.{png,jpg,gif}', lambda r: r.abort())
```

---

## 8. レンダリング済みHTMLの取得

```python
# ページ全体のHTML（JS実行後のDOM）
html = await page.content()

# 特定要素のinnerHTML
inner = await elem.inner_html()
```

---

## 9. JavaScript実行

```python
# ページ上でJSを実行
title = await page.evaluate('document.title')

# 要素に対してJSを実行
text = await elem.evaluate('el => el.textContent.trim()')

# 複雑なJS
data = await page.evaluate('''() => {
    return Array.from(document.querySelectorAll('a'))
        .map(a => a.href)
}''')
```

---

## 10. スクロール

```python
# ページ最下部までスクロール（無限スクロール対策）
await page.evaluate('window.scrollTo(0, document.body.scrollHeight)')

# 要素をビューに入れる
await elem.scroll_into_view_if_needed()
```

---

## 11. iframe

```python
# iframeの中の要素を取得
frame = page.frame_locator('iframe#target')
frame = page.frame_locator('iframe#target').nth(0)
elem  = await frame.locator('.content').element_handle()
elems  = await frame.locator('.content').element_handles()
frame.locator(".content").wait_for(timeout=10000)
```

---

## 12. 複数タブ・ポップアップ

```python
# 新しいタブが開くのを待ち受ける
async with context.expect_page() as new_page_info:
    await page.click('a[target="_blank"]')
new_page = await new_page_info.value
await new_page.wait_for_load_state()
```

---

## 13. ダイアログ処理

```python
# alert / confirm / prompt を自動で閉じる
page.on('dialog', lambda dialog: dialog.accept())
```

---

## 14. スクリーンショット（デバッグ用）

```python
# ページ全体
await page.screenshot(path='page.png', full_page=True)

# 特定要素
await elem.screenshot(path='elem.png')
```

---

## 優先度まとめ

| 優先度 | 機能 |
|---|---|
| ★★★ 必須 | ナビゲーション、要素取得、テキスト・属性取得、待機 |
| ★★☆ 頻出 | ネットワーク制御、HTML取得、JavaScript実行、スクロール |
| ★☆☆ 場面次第 | クリック・入力、iframe、複数タブ、ダイアログ |