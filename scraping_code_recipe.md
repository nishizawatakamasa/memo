# スクレイピングで使う便利関数

```py
import time
import tkinter as tk
from collections.abc import Generator
from tkinter import messagebox
from typing import NoReturn
from urllib.parse import urlencode

import requests
from requests.exceptions import InvalidSchema
from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.remote.webelement import WebElement

class Counter:
    '''カウンター。
    
    Attributes:
        _num:
            _count_upの値を格納し、スクレイピングの件数ごとに番号を振っていく。
        _count_up:
            1から順にカウントアップするジェネレータイテレータ。
    '''
    def __init__(self) -> None:
        self._num: int
        self._count_up: Generator[int, None, NoReturn]
    
    def _count_up_generator(self) -> Generator[int, None, NoReturn]:
        '''1から順にカウントアップするジェネレータ関数。'''
        count = 0
        while True:
            count += 1
            yield count

    def init_num(self) -> None:
        '''取得データ件数番号を初期化。'''
        self._num = 0
        self._count_up = self._count_up_generator()
        
    def count_up_num(self) -> None:
        '''取得データ件数番号を1からカウントアップ。'''
        self._num = next(self._count_up)

    @property
    def num(self) -> int:
        '''_count_upの値を格納し、スクレイピングの件数ごとに番号を振っていく。'''
        return self._num

def save_img(img_path: str, img_elem: WebElement | None) -> None:
    '''渡されたimg要素から画像データを取得し、pngファイルとして保存。'''
    if img_elem:
        try:
            response = requests.get(img_elem.get_attribute('src'))
        except InvalidSchema as e:
            print(f'{type(e).__name__}: {e}')
        else:
            time.sleep(1)
            with open(img_path, 'wb') as f:
                f.write(response.content)

def save_screenshot(screenshot_path: str, target_elem: WebElement | None, driver: WebDriver) -> None:
    '''渡されたWeb要素のスクリーンショットをpngファイルとして保存。'''
    if target_elem:
        driver.execute_script('arguments[0].scrollIntoView({behavior: "instant", block: "end", inline: "nearest"});', target_elem)
        time.sleep(3)
        target_elem.screenshot(screenshot_path)

def scroll_to_end():
    '''ページの最後までスクロールする。'''
    last_height = None
    while True:
        new_height = driver.execute_script('return document.body.scrollHeight')
        if new_height == last_height:
            return
        last_height = new_height
        driver.execute_script('window.scrollTo(0, document.body.scrollHeight);')
        time.sleep(3)

def pause_proc(message: str) -> None:
    '''処理を一時停止(ダイアログを表示)。'''
    root = tk.Tk()
    root.attributes('-topmost', True)
    root.withdraw()
    messagebox.showinfo(message=message)
    root.destroy()

def create_google_search_url(search_words: list[str], english_search: bool = False, search_for_images: bool = False) -> str:
    '''指定した複数のワードでgoogle検索をした結果のURLを作成。
    
    Args:
        search_words:
            検索ワードのリスト。
        english_search:
            Trueで英語検索。
        search_for_images:
            Trueで画像検索。
    '''
    query_dict = {'q': ' '.join(search_words), 'udm': '1'}
    if english_search:
        query_dict['gl'] = 'us'
        query_dict['hl'] = 'en'
    if search_for_images:
        query_dict['udm'] = '2'
    query_string = urlencode(query_dict)
    return f'https://www.google.com/search?{query_string}'

def create_google_map_search_url(search_words: list[str], english_search: bool = False) -> str:
    '''指定した複数のワードでgoogleマップ検索をした結果のURLを作成。
    
    Args:
        search_words:
            検索ワードのリスト。
        english_search:
            Trueで英語検索。
    '''
    query_dict = {'output': 'search', 'q': ' '.join(search_words)}
    if english_search:
        query_dict['gl'] = 'us'
        query_dict['hl'] = 'en'
    query_string = urlencode(query_dict)
    return f'https://maps.google.com/maps?{query_string}'

def next_hrefs1(select_next_button: Callable[[], WebElement], by_click: bool = False) -> list[str]:
    '''nextボタン要素を特定し、そのhrefを開きながら(by_click=Trueならばクリックしながら)取得していく。'''
    hrefs = [d.driver.current_url]
    while True:
        next_ = select_next_button() if by_click else d.attr('href', select_next_button())
        if next_:
            d.click(next_) if by_click else d.go_to(next_)
            hrefs.append(d.driver.current_url)
        else:
            break
    return hrefs

def next_hrefs2(select_prev_and_next_button: Callable[[], list[WebElement]], by_click: bool = False) -> list[str]:
    '''prev&nextボタン要素を特定し、nextのhrefを開きながら(by_click=Trueならばクリックしながら)取得していく。

    Note:
        *nextボタンの判別方法。\n
        1.最初はボタンが一つ。←それがnext。\n
        2.次からボタンが二つ。←二つ目がnext。\n
        3.最後にまたボタンが一つに。←それはprevだからnextは無し。
    '''
    hrefs = [d.driver.current_url]
    first_page = True
    while True:
        prev_and_next = select_prev_and_next_button() if by_click else [d.attr('href', elem) for elem in select_prev_and_next_button()]
        match len(prev_and_next):
            case 0:
                break
            case 1:
                if first_page:
                    next_ = prev_and_next[0]
                    first_page = False
                else:
                    break
            case 2:
                next_ = prev_and_next[1]
        d.click(next_) if by_click else d.go_to(next_)
        hrefs.append(d.driver.current_url)
    return hrefs

def parent(self, elem: WebElement | None) -> WebElement | None:
    '''Get parent element.'''
    return self._driver.execute_script('return arguments[0].parentElement;', elem) if elem else None

def prev_sib(self, elem: WebElement | None) -> WebElement | None:
    '''Get previous sibling element.'''
    return self._driver.execute_script('return arguments[0].previousElementSibling;', elem) if elem else None

def add_class(self, elems: list[WebElement], class_name: str) -> None:
    '''Add a class to the specified web elements.'''
    for elem in elems:
        self._driver.execute_script(f'arguments[0].classList.add("{class_name}");', elem)
```