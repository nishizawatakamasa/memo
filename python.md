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
# シーケンスに対して使う
# 指定した位置の値を取得できる。
# 指定した位置に値を代入できる。
# -は逆からになる
# 該当要素がない場合は例外が発生(IndexError)。
li[0]
# スライス
# シーケンスに対して使う
# 指定した部分のシーケンスを取得できる。
# 指定した部分にシーケンスを代入できる。
# -は逆からになる
# スライスで取得した新しいシーケンスは浅いコピーとなる
# step数に満たない部分はそのままの要素数で返る。
# 該当要素がない場合は空のシーケンスが返る。
li[start:stop:step]
li[1:-2:-1]




# 組み込み関数

# 引数の絶対値を返す。
# absolute valueの略
abs()

# 引数に渡したイテラブルオブジェクトの全要素がTrueならば、Trueを返す。
# 注意点：引数の要素が空の場合はTrueを返す。
all()

# 引数に渡したイテラブルオブジェクトの要素が一つでもTrueならば、Trueを返す。
# 注意点：引数の要素が空の場合はFalseを返す。
any()

# 引数を真偽値に変換して返す。
bool()

# 引数に指定した2つの数値(intかfloat)の商と余りをタプルとして返す。
# 引数のどちらかがfloatの場合は、商と余りもfloatになる。
# 引数のどちらかが0の場合は例外が発生する(ZeroDivisionError)。
divmod(7, 3)
# (2, 1)

# enumerate関数
# 引数に指定したイテラブルオブジェクトの要素とインデックスをタプルとして返すイテレータを作成する。
# インデックスは0から始まるが、startパラメータで変更することができる。
fruits = ['apple', 'banana', 'cherry']
for i, fruit in enumerate(fruits, start=1):
    print(i, fruit)
# 1 apple
# 2 banana
# 3 cherry

# 引数に与えられた文字列をPythonの式として評価し、その結果を返す。
# ※危険！
eval()

# 引数に与えられた文字列をPythonの文として実行する。
# ※危険！
exec()


# filter関数
# 第一引数(関数)は第二引数(イテラブルオブジェクト)の各要素に適用し、条件に合う要素だけを残します。
# 第一引数(関数)を第二引数(イテラブルオブジェクト)の各要素に適用し、条件に合う要素だけを残します。
# 結果はイテレータで返します。
関数はイテラブルオブジェクトの各要素を引数にとり、真偽値を返す必要があります。
def is_even(x):
    return x % 2 == 0

numbers = [1, 2, 3, 4, 5, 6]
evens = filter(is_even, numbers)
print(evens) # <filter object at 0x000001E8B0A9F7F0>
print(list(evens)) # [2, 4, 6]




# 引数に渡したオブジェクトの識別子(メモリアドレス)を返す。
id()

# 引数に渡した文字列を標準出力に表示し、ユーザーからの入力を受け取って返す。
# 引数がない場合は、何も表示しない。
input()

# オブジェクトの型を判定する。
# 引数に渡したオブジェクトが指定したクラスのインスタンスであるかどうかを真偽値で返す。
isinstance(s, str)

# 引数に渡したイテラブルオブジェクトをイテレータに変換して返す。
iter()

# 引数に渡したイテラブルオブジェクトの長さ(要素数や文字数)を返す。
len()


# map関数
# 第一引数(関数)を第二引数(イテラブルオブジェクト)の各要素に適用した結果をイテレータで返します。
def square(x):
    return x ** 2

arr = [1, 2, 3, 4, 5]
result = map(square, arr)
print(result) # <map object at 0x000001E8F1A6B9A0>
print(list(result)) # [1, 4, 9, 16, 25]





max()
memoryview()
min()


# 引数に渡したイテレータの次の要素を返す。
next()

O
object()

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
[組み込み関数全71件 完全解説](https://qiita.com/t_aki/items/a5e578aecf8cc20bec31)  
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