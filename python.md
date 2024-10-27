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
# リストのみミュータブル（可変）。他は全てイミュータブル（不変）。
# 順番が保証されている。
# インデックス、スライスによる取得が可能。リストのみ代入も可能。

# リスト
list()
# タプル
# タプルを作るのはカンマであり、丸括弧ではない。
# そのため、丸括弧は省略可能で要素が1個の場合は末尾にカンマが必要。
tuple()
# 文字列
str()
# rangeオブジェクト
range(start, stop, step)

# マッピング
# ミュータブル（可変）
# 順番が保証されている(Python 3.7以降)。

# 辞書
dict()

# 集合
# ミュータブル（可変）
# 順序をもたない。

# 集合
# 一意な値のみが要素として残る。
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


# 三項演算子
# 真のときに返す値 if 条件式 else 偽のときに返す値


# 内包表記
# リスト内包表記を例にとる
# 基本形
[i for i in iterable]
# ifで条件分岐することもできる。以下のように後置でifを記述する。
# ifの条件式がTrueとなる要素のみ式で評価される。
# Falseとなる要素は無視される。
li = [i * 2 for i in range(10) if i % 2 == 0]
# 三項演算子と組み合わせることもできる。
li = [i * 2 if i % 2 == 1 else 0 for i in range(10)]
# セット内包表記
num_set = {i * 2 for i in range(10)}
# 辞書内包表記
fruits = ['apple', 'banana', 'lemon']
prices = [180, 100, 85]
di = {fruit: price for fruit, price in zip(fruits, prices)}
# ジェネレータ式
gen = (i * 2 for i in range(10))
# タプル内包表記は存在しないが、tuple()関数を使用して以下のように書く方法がある。
t = tuple(i for i in range(10))



# 組み込み関数

# 引数(イテラブルオブジェクト)からリストを作成する。
# 引数がない場合は空のリストを返す。
list()
# 引数(イテラブルオブジェクト)からタプルを作成する。
# 引数がない場合は空のタプルを返す。
tuple()
# 引数(オブジェクト)を文字列に変換する。
# 引数がない場合は空文字を返す。
str()
# rangeオブジェクトを作成する。
range(start, stop, step)
# 引数に指定されたキーと値のペアから辞書を作成する。
# 引数がない場合は空の辞書を返す。
dict()
# 引数(イテラブルオブジェクト)から集合を作成する。
# 引数がない場合は空の集合を返す。
set()

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
# 第一引数(関数)を第二引数(イテラブルオブジェクト)の各要素に適用し、条件に合う要素だけを残す。
# 関数はイテラブルオブジェクトの各要素を引数にとり、真偽値を返す必要がある。
# 結果はイテレータで返す。
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


# 引数(イテラブルオブジェクト)の中で最大の要素を返す。
# 引数(イテラブルオブジェクト)が空の場合にはValueErrorを発生させる。
max()

# 引数(イテラブルオブジェクト)の中で最小の要素を返す。
# 引数(イテラブルオブジェクト)が空の場合にはValueErrorを発生させる。
min()

# 引数に渡したイテレータの次の要素を返す。
next()

# 引数(ファイル名)を開いてファイルオブジェクトを返す。
# ファイルを開く際には、必ずcloseメソッドを呼び出してファイルを閉じる必要がある。
# with文を使うと、ファイルを自動的に閉じることができる。
open()

# 第一引数(数値)を第二引数(数値)乗した結果を返します。
pow()

# 引数(任意のオブジェクト)を標準出力に表示する。
print()

# 引数に渡したオブジェクトを、eval()で再び元のオブジェクトに戻せる文字列に変換して返す。
# representationの略。
# デバッグやログ出力などに使われる。
repr()

引数(シーケンスオブジェクト)の逆順のイテレータを返す。
reversed()

# 第一引数(数値)を第二引数(整数)で指定した桁数に丸めた結果を返す。
# 第二引数を指定しなければ、整数に丸める。
# 第二引数が負の場合には、10のべき乗の位に丸める。
# 端数がちょうど0.5なら、「切り捨て」と「切り上げ」のうち結果が偶数となる方へ丸める(偶数への丸め)。
print(round(3.14159, 2)) # 3.14
print(round(3.14159)) # 3
print(round(12345, -2)) # 12300

# スライスオブジェクトを生成できる。
slice(start, stop, step) # 引数を3つ指定した場合
slice(start, stop) # 引数を2つ指定した場合
slice(stop) # 引数を1つ指定した場合
# 使い方
sl = slice(start, stop, step)
l = [0, 10, 20, 30, 40, 50, 60]
l[sl]

# 引数(イテラブルオブジェクト)の要素をソートした新しいリストを返す。
l = [3, 1, 4, 5, 2]
l_sorted = sorted(l)
# デフォルトは昇順。降順にソートしたい場合は引数reverseをTrueとする。
l_reverse_sorted = sorted(l, reverse=True)

# 引数(イテラブルオブジェクト)の要素の合計を返す。
# 第二引数に初期値を指定できる。
# 引数の要素は全て数値である必要がある(そうでない場合はTypeErrorが発生する)。
print(sum([1, 2, 3], 10)) # 16

# 親クラスのオブジェクトを返す。
# 親クラスのオブジェクトは、サブクラスのオブジェクトから親クラスの属性やメソッドにアクセスするために使われる。
super()

# 引数(オブジェクト)の型を返す。
# 引数が三つの場合は新しい型を作成する。
type()

# 複数のイテラブルオブジェクトを引数にとり、それらの要素をまとめたイテレータを返す。
# イテレータの要素は、引数のイテラブルオブジェクトの要素を対応させたタプル。
# 引数のイテラブルオブジェクトの長さが異なる場合、最も短いイテラブルオブジェクトが終了するまで要素をまとめる。残りの要素は無視される。
a = [1, 2, 3, 4]
b = ['a', 'b', 'c']
c = zip(a, b) # イテレータを返す
print(list(c)) # [(1, 'a'), (2, 'b'), (3, 'c')]











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