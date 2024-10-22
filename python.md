# Python覚書

* 目次
    * [基本](#基本)
    * [venvを使った仮想環境の作成](#venvを使った仮想環境の作成)
    * [settings.jsonにモジュールの検索パスを追加する](#settings.jsonにモジュールの検索パスを追加する)
    * [抽象基底クラス](#抽象基底クラス)
    * [参考サイト](#参考サイト)



<a id="基本"></a>
## 基本


```py





# 数値型
# 整数
int()
# 少数
float()



# イテラブルオブジェクト
# イテラブルオブジェクトはイテレータオブジェクトを生成できるオブジェクト
# イテレータオブジェクトは要素を１つずつ取り出せるオブジェクト

# シーケンス

# リスト
# ミュータブル（可変）
# インデックス、スライスで取得と代入が可能
# 順番が保証されている。
list()
# タプル
# イミュータブル（不変）
# インデックス、スライスでは取得のみ可能
# タプルを作るのはカンマであり、丸括弧ではない。
# そのため、丸括弧は省略可能で要素が1個の場合は末尾にカンマが必要。
# 順番が保証されている。
tuple()
# 文字列
# イミュータブル（不変）
# インデックス、スライスでは取得のみ可能
# 順番が保証されている。
str()
# rangeオブジェクト
# イミュータブル（不変）
# インデックス、スライスでは取得のみ可能
# 順番が保証されている。
range(start, stop, step)

# マッピング

# 辞書
# ミュータブル（可変）
# 順番が保証されている(Python 3.7以降)。
dict()

# 集合

# 集合
# ミュータブル（可変）
# 一意な値のみが要素として残る。
# 順序をもたない。
set()









# インデックス
# 指定した位置の値を取得。
# 指定した位置に値を代入。
li[0]
# スライス
# 指定した部分のシーケンスを取得。
# 指定した部分にシーケンスを代入。
li[start:stop:step]
li[1:-2:-1]
# -は逆からになる
# スライスで取得した新しいシーケンスは浅いコピーとなる

# 該当要素がない場合はIndexError
該当要素が一つもなくても空のシーケンスを返せる
step数に満たない部分はそのままの要素数で返る





引数の絶対値を返す。
absolute valueの略
abs()



all()
anext()
any()
ascii()

B
bin()
bool()
breakpoint()
bytearray()
bytes()

C
callable()
chr()
classmethod()
compile()
complex()

D
delattr()
dict()
dir()
divmod()

E
enumerate()
eval()
exec()

F
filter()
float()
format()
frozenset()

G
getattr()
globals()

H
hasattr()
hash()
help()
hex()

I
id()
input()
int()
isinstance()
issubclass()
iter()
L
len()
list()
locals()

M
map()
max()
memoryview()
min()

N
next()

O
object()
oct()
open()
ord()

P
pow()
print()
property()




R
range()
repr()
reversed()
round()

S
set()
setattr()
slice()
sorted()
staticmethod()
str()
sum()
super()

T
tuple()
type()

V
vars()

Z
zip()

_










7. 単純文 (simple statement)
7.1. 式文 (expression statement)
7.2. 代入文 (assignment statement)

# アサーションの条件がTrueの場合は何も起きず、Falseの場合は例外が発生(AssertionError)。
assert 0 < hoge



7.4. pass 文
7.5. del 文
7.6. return 文
7.7. yield 文
7.8. raise 文
7.9. break 文
7.10. continue 文
7.11. import 文
7.12. global 文
7.13. nonlocal 文

8. 複合文 (compound statement)
8.1. if 文
8.2. while 文
8.3. for 文
8.4. try 文
8.5. with 文
8.6. match 文
8.7. 関数定義
8.8. クラス定義

# 定数
COLMUN_NAMES

zip()
enumerate()
range(start, stop, step)
len()
内包表記（list,dict）
['' for _ in range(7)] if句付きも
f'{v}'

辞書内包表記
集合内包表記

# アンパック
*li

# スライス
# スライスではインデックスをオーバーしてもエラーにならない
[::]
# 演算子
+= //

三項演算子

with

li.append(COLMUN_NAMES)
li.extend(body)

差集合

コンテキストマネージャ
コンテキストマネージャは、リソースの初期化と解放を自動的に管理するための構文です。with ステートメントを使って、ファイルの操作やデータベース接続などで使用されます。
```




### 参考サイト
[Python基礎文法まとめ](https://qiita.com/kita_ds12/items/84552d41a8aad36d5519)  
[Pythonの基本的な組み込み型とその一般論まとめ](https://qiita.com/nakasan/items/bc9ba8eb57f5b7a22698)  
[Pythonのzip関数の罠を回避する](https://zenn.dev/nakurei/articles/avoiding-python-zip-function-trap)



<a id="venvを使った仮想環境の作成"></a>
## venvを使った仮想環境の作成

exe化したいファイルがあるディレクトリで  
`python -m venv env`  

仮想環境の有効化  
`$ source env/Scripts/activate`   

仮想環境の無効化  
`deactivate`  

モジュールをファイルに記載して管理  
`pip freeze > requirements.txt`  

モジュールの一括インストール(カレントディレクトリで)  
`pip install -r requirements.txt`  

仮想環境の削除  
envフォルダを削除するだけ  

スクリプトの実行を許可。ExecutionPolicy  
settings.jsonに設定
```json:settings.json
"terminal.integrated.env.windows": {
  "PSExecutionPolicyPreference": "RemoteSigned"
},
```
<a id="settings.jsonにモジュールの検索パスを追加する"></a>
## settings.jsonにモジュールの検索パスを追加する

自動取得の対象とならない場合、明示的な設定が必要となる。  
設定例
```json:settings.json
"python.analysis.extraPaths": ["/Users/user/AppData/Local/programs/Python/Python312/lib/site-packages"]
```
<a id="抽象基底クラス"></a>
## 抽象基底クラス

```python
from abc import ABCMeta, abstractmethod

class BaseSe(metaclass=ABCMeta):

    @abstractmethod
    def hoge(self):
      pass
```

<a id="参考サイト"></a>
## 参考サイト

[Pythonプログラミング入門](https://utokyo-ipp.github.io/index.html)