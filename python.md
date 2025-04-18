# Python覚書

* 目次
    * [特徴](#特徴)
    * [基本](#基本)
    * [変数のスコープ](#変数のスコープ)
    * [よく使われるモジュール（ライブラリ）](#よく使われるモジュール（ライブラリ）)
    * [よく使われる関数](#よく使われる関数)
    * [reを使った正規表現操作](#reを使った正規表現操作)
    * [pywebviewを使ったデスクトップアプリの作成](#pywebviewを使ったデスクトップアプリの作成)
    * [venvを使った仮想環境の作成](#venvを使った仮想環境の作成)
    * [Pyinstallerを使ったスクリプトのexe化](#Pyinstallerを使ったスクリプトのexe化)
    * [pipコマンド](#pipコマンド)
    * [Flitを使ったPyPIへのパッケージ公開](#Flitを使ったPyPIへのパッケージ公開)
    * [settings.jsonにモジュールの検索パスを追加する](#settings.jsonにモジュールの検索パスを追加する)
    * [抽象基底クラス](#抽象基底クラス)
    * [参考サイト](#参考サイト)


<a id="特徴"></a>
## 特徴

短く少ないコードで多くのことを実現する、という美学がある。  
なんでもできるオールラウンダーな言語だが、他の言語のほうが適している分野も多々ある。  
Pythonの真骨頂といえる分野はデータ処理と自動化。    
データ処理と自動化は密接に関わり合っており、切り離して考えるものではない。

データ処理の簡単な分類
* データ収集
* データ操作
* データ可視化
* データ学習


<a id="基本"></a>
## 基本

```py

# リテラル
# Pythonのプログラムに直接記述された値(定数)。

#組み込み定数
True # 真を表すbool型の値。整数に変換すると1になる。
False # 偽を表すbool型の値。整数に変換すると0になる。
None # 存在しないことを表すNoneType型の値。

# 数値リテラル
123000 # 整数
3.14 # 浮動小数点数
# 数字の間に_を入れ, 外見上のみ桁をグループ化することができる。
100_000_000
3.14_15_92

# 文字列リテラル
'hoge'
"fuga"
'''
piyo
'''

# raw文字列
# Pythonの文字列のエスケープシーケンスを無効化する。
# ※注意：正規表現内のエスケープシーケンスは無効化しない(\dとか\]とか)。
r'hoge'


# 代入

a = 100
a = b = c = 'string'
a, b, c, d = d, c, b, a
l = [0, 1, 2, 3, 4] # リストの要素の値を入れ替え（並べ替え）
l[0], l[3] = l[3], l[0]
a := b # セイウチ演算子。代入と同時に値も返す。つまり、「変数への代入」と「評価」をまとめて行える。

# 同時代入
# イテラブルオブジェクトの各要素を複数の変数に同時代入できる。
# 変数の数と要素の数が一致していないとValueErrorが発生する。
a, b, c = 0.1, 100, 'string'
*a, b = [100, 200, 300] # 変数に*をつけることで残りをリストとして代入できる(*がつけられるのは一つだけ)。
a, b, c, *d = (0, 1, 2) # 余分の要素がない場合は空のリストが代入される。
print(d) # []

a, b, (c, d, e) = (0, 1, (2, 3, 4)) # 左辺の変数を()または[]で囲むとアンパックできる。
a, b, *_ = (0, 1, 2, 3, 4) # 必要のない値は慣例的にアンダースコア_に代入



# 展開（アンパック）

# イテラブルオブジェクト、dict_keys、dict_values、dict_itemsクラスに*を付けると、値を展開できる。
*li
*di
*d.keys()
*d.values()
*d.items()


# 関数の引数に展開（アンパック）した値を渡す

# 展開した値を引数に指定すると、それぞれの値が個別の引数として渡される。
# dictに**を付けて引数に指定すると、要素のキーが引数名、値が引数の値として展開されて、それぞれ個別の引数として渡される。
hoge = fuga(*li, **di)


# 可変長引数

# 関数定義で引数に*をつけると、複数の引数をタプルとして受け取る。
# 関数定義で引数に**をつけると、複数のキーワード引数を辞書として受け取る。
# 慣例として*args, **kwargsという名前が使われることが多い。
def hoge(*args, **kwargs):
    pass



# 算術演算子
# 足し算
10 + 3 # 13
# 引き算
10 - 3 # 7
# 掛け算
10 * 3 # 30
# 割り算
10 / 3 # 3.3333333333333335
# 割り算の整数部
10 // 3 # 3
# 割り算の余り
10 % 3 # 1
# べき乗
10**3 # 1000

# 0での割り算はエラーZeroDivisionErrorとなる。

# 累算代入演算子
# 左辺のオブジェクトに処理結果が代入、更新される。
a += 5 - 2 # a = a + (5 - 2)
a -= 5 - 2 # a = a - (5 - 2)
a *= 5 - 2 # a = a * (5 - 2)
a /= 5 - 2 # a = a / (5 - 2)
a //= 5 - 2 # a = a // (5 - 2)
a %= 5 - 2 # a = a % (5 - 2)
a **= 5 - 2 # a = a ** (5 - 2)

# +演算子は、リスト同士、タプル同士、文字列同士を連結することもできる。
# 連結には累算代入演算子+=も使える。
# 特殊ケース：リストに+=を使った場合は、li.extend(iterable)と同じ挙動となる。つまり、連結するイテラブルオブジェクトの型は問わない。
# |演算子で辞書同士を連結することもできる(Python 3.9以降)。例：d3 = d1 | d2
# 連結には累算代入演算子|=も使える。


# *演算子を使って、リスト、タプル、文字列と整数をかけると、要素が繰り返しとなる。
l = [1, 10, 100]
print(l * 3) # [1, 10, 100, 1, 10, 100, 1, 10, 100]
# 整数値が先でも同じ結果を返す。
print(3 * l) # [1, 10, 100, 1, 10, 100, 1, 10, 100]
# 整数値が負の場合、空のリスト、タプル、文字列を返す。
print(l * -1) # []
# 整数int以外の型の場合、TypeErrorが発生する。
# 累算代入演算子*=も使える。
l *= 3
print(l) # [1, 10, 100, 1, 10, 100, 1, 10, 100]

# 比較演算子
a == b # aがbと同値。※基本的には厳密比較だが、数値型は一括り(ちなみにbool型はint型のサブクラスであるため、数値型に含まれる。Falseは0、Trueは1)。
a != b # aがbと同値でない。
a < b # aがbよりも小さい。
a > b # aがbよりも大きい。
a <= b # aがb以下である。
a >= b # aがb以上である。
a is b # aがbと同一。主にNoneの判定に使う。。
a is not b # aがbと同一でない。
a in b # aがbに含まれる(bはイテラブルオブジェクト)。
a not in b # aがbに含まれない(bはイテラブルオブジェクト)。
2 < a <= 10 # ※連結することもできる。

# 論理演算子
and # 論理積（かつ）
or # 論理和（または）
not # 否定（でない）


# 演算子の優先順位（計算順序）

# べき乗
**
# 正数、負数
+x, -x
# 掛け算と割り算
*, /, //, %
# 足し算と引き算
+, -
# 比較演算子
in, not in, is, is not, <, <=, >, >=, !=, ==
# 論理演算子 NOT
not
# 論理演算子 AND
and
# 論理演算子 OR
or


# Pythonにおいては全てがオブジェクトである。
# int()、str()、list()などは型、つまりクラスだが、一般的な型のためあえて全て小文字の命名になっている。

# 数値型

# 整数
int()
# 小数
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
# 空タプルの生成には丸括弧が必須だが、その場合は引数なしのtuple()で生成するほうが分かりやすいと思う。
tuple()
# 文字列
str()
# rangeオブジェクト
range(start, stop, step)

# マッピング
# ミュータブル（可変）
# 順番が保証されている(Python 3.7以降)。

# 辞書
# キーに使えるのはイミュータブル（不変）なデータのみ。
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
# 該当要素がない場合はエラーが発生(IndexError)。
li[0]
# スライス
# シーケンスに対して使う
# 指定した部分のシーケンスを取得できる。
# 指定した部分にシーケンスを代入できる。
# -は逆からになる
# スライスで取得した新しいシーケンスは浅いコピーとなる
# step数に満たない部分はそのままの要素数で返る。
# 該当要素がない場合は空のシーケンスが返る(エラーは発生しない)。
li[start:stop:step]
li[1:-2:-1]



# 辞書に含まれる全てのキーや値を取得する方法
# dictのkeys(), values(), items()メソッドを使う

# 全てのキーを取得する。
# dict_keysクラスを返す。
di.keys()
# 全ての値を取得する。
# dict_valuesクラスを返す。
di.values()
# 全てのキーと値を(タプルで)取得する。
# dict_itemsクラスを返す。
di.items()

# dict_keys、dict_values、dict_itemsクラスについて。

# in演算子を使える。
# for文でループ処理できる。
# *でアンパックできる。
# list()でリスト化できる(dict_itemsの場合は各要素が(key, value)のタプルとなる)。
# dict_values以外は集合演算をすることが可能(valueは重複する場合があるため)。



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

# 現在のスコープにある変数の辞書を返す。
locals()

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




# 単純文 (simple statement)

# アサーションの条件がTrueの場合は何も起きず、Falseの場合は例外が発生(AssertionError)。
assert a > 0
assert a > 0, 'aは正の値でなければなりません'

# 何も起きない(文が必要だが何も実行したくない場合のプレースホルダとして有用)。
pass

# 指定したオブジェクトを削除する(リストは再帰的に削除される)。
del a

# 指定した値を戻り値として返し、関数の処理を終了する。
return a

# 指定した値を戻り値として返し、関数の処理を一時停止する。
# 停止位置を記憶しており、再実行時はそこから再開される。
# 関数定義内でyieldを使用することで、その定義は通常の関数でなくジェネレータ関数になる。
yield a

# 明示的に例外を発生させる。
raise ValueError
raise ValueError('入力値が不正です')

# ループ処理を抜ける。
# ループの内側でのみ出現することができる(ループ内の関数定義やクラス定義の内側には出現できない)。
break

# 現在のループを一回分スキップする。
# ループの内側でのみ出現することができる(ループ内の関数定義やクラス定義の内側には出現できない)。
continue

# モジュールやクラス、関数などをインポートする
import time
# from 形式
from time import sleep

# 列挙した識別子を、グローバル変数として解釈するよう指定する。
global a, b

# 列挙した識別子を、一つ外側のスコープ(グローバルを除く)の変数として解釈するよう指定する。
nonlocal a, b, c

# 型エイリアスを定義する。
type Number = int | float


# 複合文 (compound statement)

# if文
if condition1:
    # condition1が真の時の処理
elif condition2:
    # condition1が偽でcondition2が真の時の処理
else:
    # 全ての条件が偽の時の処理

# while文
while some_condition:
    # some_conditionが真の間、繰り返し実行する処理。
    if condition:
        break
else:
    # ループが中断されずに終了した場合に実行される処理
    # ※考え方：conditionが真ならばbreakする。conditionがすべて偽ならば、breakせずelse節を実行する。
    # if: break else: ~

# for文
for item in iterable:
    # iterableの要素を順番にitemに代入しながら、繰り返し実行される処理
    if condition:
        break
else:
    # ループが中断されずに終了した場合に実行される処理
    # ※考え方：conditionが真ならばbreakする。conditionがすべて偽ならば、breakせずelse節を実行する。
    # if: break else: ~

# try文
try:
    # 例外が発生するかもしれない処理
except SomeException:
    # 指定した例外が発生したら実行される処理
except OtherException:
    # except節は複数指定できる。複数の例外に対してそれぞれ異なる処理を記述可能。
except (ZeroDivisionError, ValueError) as e:
  print("エラーが発生しました:", e)
    # except節に複数の例外を指定することもできる。
    # 例外オブジェクトをasで受けることもできる。
else:
    # 例外が発生しなかった場合に実行される処理
    # ※考え方：except(例外が発生したら) ~ else(例外が発生しなかったら)
finally:
    # 例外の有無に関わらず必ず実行される処理

# with文
# コンテキストマネージャ(型)を起動し、生成されたオブジェクトを変数targetに代入して利用する。
# コンテキストマネージャとは、__enter__()と__exit__()の2つのメソッドを持つ型である。
with ContextManager() as target:
    # targetを使った主処理

# ※処理の順番は以下
__init__()
__enter__() # ここでreturnしたオブジェクトが、変数targetに代入される。通常はself。
# targetを使った主処理
__exit__() # 例外の有無に関わらず必ず実行される

# open()で返したファイルオブジェクトはコンテキストマネージャ型であるため、with文が使える。



# match文
# 値が一致するかどうかでマッチングを行う。
# 最初にマッチしたcase文のみが実行され、match文を抜ける。 
match a:
    case 1:
        print('aは数値の1')
    case '1':
        print('aは文字列の1')
    # or条件
    case 2 | 3:
        print('aは2か3')
    # ifで条件を追加できる(ifガード)
    case 5 if b == 1:
        print('aは5(bは1)')
    case 5 if b == 2:
        print('aは5(bは2)')
    # 何にでもマッチするワイルドカードパターン
    # 必ず最後に置く必要がある(でないとSyntaxErrorが発生)
    case _:
        print('aはその他')



# リストのメソッド

# リストの末尾（最後）に要素を追加する。
li.append(x)
# リストにイテラブルオブジェクト(型を問わず)を連結する。
li.extend(iterable)
# リストの末尾（最後）の要素を取り除いて返す(リストが空の場合はIndexErrorが発生)。
li.pop()
# リスト内でのxの出現回数をintで返す。
li.count(x)


# 辞書のメソッド

# 第一引数にキーを指定。
# キーが存在する場合は対応する値が返る。
# キーが存在しない場合は第二引数に指定したデフォルト値を返す(省略時はNone)。
di.get('key', 'キーが存在しません')

# ※参考：インデックスによるアクセスでは、キーが存在しない場合KeyErrorが発生する。
di['key']


# 集合のメソッド

# 集合に要素を追加する。
set_01.add(x)


# 比較演算
{1, 2, 3} == {1, 2, 3} # True
{1, 2, 3} != {1, 2, 3} # False
# <= や < は、集合間の包含関係を判定する。
{1, 2, 3} <= {1, 2, 3} # True
{1, 2, 3} < {1, 2, 3} # False
{1, 2} < {1, 2, 3} # True
{1, 2} <= {2, 3, 4} # False


# 集合演算
# 対象とする集合は三つ以上でもOK。
a = set('abcd')
b = set('cdef')
# 和集合(少なくともどちらか一つに含まれる要素の集合)を返す。
a | b # {'a', 'b', 'c', 'd', 'e', 'f'}
# 積集合(両方に含まれる要素の集合)を返す。
a & b # {'c', 'd'}
# 差集合(aに含まれbに含まれない要素の集合)を返す。
a - b # {'a', 'b'}
# 対称差集合(どちらか一つにのみ含まれる要素の集合)を返す。
a ^ b # {'a', 'b', 'e', 'f'}



# 関数定義時のデフォルト引数に関する注意点
# デフォルト引数の式は関数が定義されるときにただ一度だけ評価される。
# 以下はNoneを使って判定し、引数が省略されている場合に関数内で新しいオブジェクトを生成する方法。
# こうすると、関数が呼び出されるタイミングで毎回新たなオブジェクトが生成される。
def func_default_list_none(l=None, v=3):
    if l is None:
        l = []
    l.append(v)
    return l
```

### 参考サイト
[コーディングスタイルガイド](https://google.github.io/styleguide/pyguide.html)
[組み込み関数全71件 完全解説](https://qiita.com/t_aki/items/a5e578aecf8cc20bec31)  
[Python基礎文法まとめ](https://qiita.com/kita_ds12/items/84552d41a8aad36d5519)  
[Pythonの基本的な組み込み型とその一般論まとめ](https://qiita.com/nakasan/items/bc9ba8eb57f5b7a22698)  
[Pythonのzip関数の罠を回避する](https://zenn.dev/nakurei/articles/avoiding-python-zip-function-trap)



<a id="変数のスコープ"></a>
## 変数のスコープ

大きい順に
* グローバル変数
* クラス変数
* インスタンス変数
* ローカル変数

※関数(メソッド)の引数に参照の値を渡した時の挙動は、その関数内のローカル変数に参照の値を代入した時と同じ。  
※Pythonにはブロックスコープがないため、for文で定義されるループ変数も特別なものではない(単にそのスコープ内で定義された変数と同じ)。
※ただし、リスト内包表記のループ変数は内包表記外にリークしない。代入式(セイウチ演算子)は内包表記外にリークする。

<a id="よく使われるモジュール（ライブラリ）"></a>
## よく使われるモジュール（ライブラリ）


* Requests (HTTPリクエストを送信、ウェブサイトからデータを取得。)
* Beautiful Soup (HTMLを解析、要素を抽出。)
* Selenium (ブラウザの自動操作。動的ページからデータを取得。)

* NumPy (高速な数値計算。)
* Pandas (データ操作。)
* Matplotlib (静的なグラフを作成。)
* Plotly (動的なグラフを作成。)
* scikit-learn (機械学習。)

* openpyxl (Excelを操作。)
* pynput (キーボードやマウスを操作。)

* os (基本的なファイル操作。)
* shutil (高度なファイル操作。)
* glob (ワイルドカードを用いて、ファイルパスの一覧を取得できる。)
* dill (Pythonオブジェクトをバイナリ形式で保存したり、読み込んだりできる。)
* json (JSONファイルの読み込み・書き込みができる。)
* datetime (日付や時刻の操作。)




<a id="よく使われる関数"></a>
## よく使われる関数

```py

# 指定した秒数、処理を一時停止する。
time.sleep(0.001)

# プログラムを途中終了する。
sys.exit()


# パス系

# 引数に渡した複数の文字列を結合し、パス文字列にして返す。
os.path.join('main', 'log', 'sample.log')
# ファイルの存在確認。bool値を返す。
os.path.isfile()
# ディレクトリの存在確認。bool値を返す。
os.path.isdir()
# パスの存在確認。bool値を返す。
os.path.exists()



# ディレクトリ（フォルダ）を作成
# 末端ディレクトリまで、中間ディレクトリも含めて一気に新規作成できる。
os.makedirs('./temp/dir1')

# ファイルを削除
os.remove('./dir/sample_01.txt')
# 空のディレクトリを削除
os.rmdir('./temp/dir')
# ディレクトリを中身ごと削除
shutil.rmtree('./temp/dir')

# ファイルをコピー
# 第一引数はソースファイルのパス。第二引数はコピーファイルのパス
shutil.copyfile('./temp/dir1/test1.txt', './temp/dir2/test2.txt')
# ディレクトリを中身ごとコピー
# 第一引数はソースディレクトリのパス。第二引数はコピーディレクトリのパス
shutil.copytree('temp/dir1', 'temp/dir2/new_dir')

# ファイル、ディレクトリの名前を変更。
# 第一引数は変更前のパス、第二引数は変更後のパス。
os.rename('./dir/サンプル_01.txt', './dir/sample_01.txt') 
# ファイル、ディレクトリ(中身ごと)を移動
# 第一引数は移動元のパス、第二引数は移動先のパス。
# ファイルの場合、移動先に存在しない中間ディレクトリがあるとエラーになる。
# ディレクトリの場合は存在しない中間ディレクトリも自動的に作成される。
shutil.move('temp/dir1/file.txt', 'temp/dir2')
# ※os.renameとshutil.moveは、どちらも移動と名前の変更を同時に行えるメソッドだが、二つを移動用と名前の変更用に使い分けたほうが分かりやすい。



# 条件を満たすパスの文字列を要素とするリストを返す。
# 第一引数にパスの文字列を指定する。
# ワイルドカード*などの特殊文字が使用可能。
# *は長さ0文字以上の任意の文字列にマッチ。
# ?は任意の一文字にマッチ。
# []で囲むと括弧内の文字列のどれか一文字にマッチ(!をつけると括弧内にない文字にマッチ)。
# []内でハイフン(-)を使って連続する文字を指定することも可能。
# *, ?, [を特殊文字としてではなくその文字自体として使うには[]で囲む。
glob.glob('./img/*_???[aZ1][0-9][!a-z][[][?].png')
# 条件を満たすパスの文字列を順に生成するイテレーターを返す。
# それ以外はglob.glob()と同じ。
glob.iglob()

```


<a id="reを使った正規表現操作"></a>
## reを使った正規表現操作

### グループの設定
|||同パターン中での後方参照|replパターン中での参照|
|-|-|-|-|
|(?: )|グループを設定。|不可|不可|
|( )|キャプチャグループを設定。各グループに番号付き。|\1|\1|
|(?P\<name> )|キャプチャグループを設定。各グループに番号＆名前付き。|\1, (?P=name)|\1, \g\<name>|


### 正規表現における先読みと後読み
(?<= )pattern(?= )肯定  
(?<! )pattern(?! )否定


<a id="pywebviewを使ったデスクトップアプリの作成"></a>
## pywebviewを使ったデスクトップアプリの作成

pywebviewで作成したデスクトップアプリは、内部的にはWebアプリのようなことをやっている。

* Webアプリと似てるところ
    * UIの表示にWebブラウザエンジンを使っている。
    * フロントエンド（HTML, CSS, JavaScript）とバックエンド(Python)でやりとりをする。
* Webアプリと違うところ
    * デスクトップ環境で実行される。
    * リモートサーバーだけでなく、ローカルサーバーともやりとりができる。


例：
```py

import webview

# APIクラスには、JavaScriptから呼び出すことができる関数を定義する。
class Api:
    def say_hello(self, name):
        return f"Hello, {name}!"

if __name__ == '__main__':
    api = Api()
    # ウィンドウを作成
    window = webview.create_window('Hello world', 'https://example.com', js_api=api)
    # ウィンドウを表示
    webview.start()

```

```html
<script>
    // Webビュー内で動作するJavaScriptからPython側のAPI関数を呼び出すことができる。
    const hello = pywebview.api.say_hello()
</script>
```


```py
# ネイティブGUIを使用してウェブビューウィンドウを作成する。
# この関数が呼び出された後、実行はブロックされる。
# プログラム・ロジックは別のスレッドで実行する必要がある。
def create_window(
    # タイトル ウィンドウのタイトル
    title: str,
    # url： ロードするURL
    url: str | None = None,
    # html： 読み込むHTMLコンテンツ
    html: str | None = None,
    # APIクラスのインスタンス
    js_api: Any = None,
    # width: ウィンドウの幅。デフォルトは800px
    width: int = 800,
    # height: ウィンドウの高さ。デフォルトは600px。
    height: int = 600,
    x: int | None = None,
    y: int | None = None,
    # screen： ウィンドウを表示するスクリーン。
    screen: Screen = None,
    # resizable： ウィンドウのサイズを変更できる場合はTrue、そうでない場合はFalse。デフォルトはTrue
    resizable: bool = True,
    # fullscreen： フルスクリーンモードで開始する場合はTrue。デフォルトはFalse
    fullscreen: bool = False,
    # min_size: 最小ウィンドウサイズを指定する (width, height) タプル。デフォルトは200x100
    min_size: tuple[int, int] = (200, 100),
    # hidden: ウィンドウを隠すかどうか
    hidden: bool = False,
    # frameless： ウィンドウがフレームを持つかどうか。
    frameless: bool = False,
    # easy_drag： ウィンドウがフレームレスの場合の簡単なウィンドウ・ドラッグ・モード。
    easy_drag: bool = True,
    # shadow: ウィンドウに枠線をつけるかどうか (影と Windows の丸いエッジ)。
    shadow: bool = True,
    # focus： ユーザーがウィンドウを開いたときにアクティブにするかどうか。ウィンドウはマウスで操作できますが、キーボード入力はこのウィンドウではなく別の（アクティブな）ウィンドウに送られます。
    focus: bool = True,
    # 最小化： ウィンドウを最小化して表示します。
    minimized: bool = False,
    # 最大化： ウィンドウを最大化する。
    maximized: bool = False,
    # on_top： ウィンドウを他のウィンドウより上に表示する (必須 OS: Windows)
    on_top: bool = False,
    # confirm_close： ウィンドウを閉じる確認ダイアログを表示する。デフォルトは False
    confirm_close: bool = False,
    # background_color: ウェブビューのコンテンツが読み込まれる前に表示される背景色。デフォルトは白。
    background_color: str = '#FFFFFF',
    # transparent： ウィンドウの背景を描画しない。
    transparent: bool = False,
    # text_select： ページ上でのテキスト選択を許可する。デフォルトは False。
    text_select: bool = False,
    zoomable: bool = False,
    draggable: bool = False,
    vibrancy: bool = False,
    localization: Mapping[str, str] | None = None,
    # サーバー： サーバークラス。デフォルトはBottleServer
    server: type[http.ServerType] = http.BottleServer,
    http_port: int | None = None,
    # server_args： サーバーのインスタンス化に渡す引数の辞書。
    server_args: http.ServerArgs = {},
# :return: ウィンドウオブジェクト。
) -> Window:


# GUIループを開始し、以前に作成したウィンドウを表示する。
# この関数はメインスレッドから呼び出されなければならない。
def start(
    # func： GUIループの開始時に呼び出す関数。
    func: Callable[..., None] | None = None,
    # args: 関数の引数。単一の値、または値のタプル
    args: Iterable[Any] | None = None,
    # localization： ローカライズされた文字列を含む辞書。デフォルトの文字列とそのキーは localization.py で定義されています。
    localization: dict[str, str] = {},
    # gui： 特定のGUIを強制する。指定できる値は ``cef``, ``qt`` です。
    # プラットフォームによって ``cef``、``qt``、``gtk``、``mshtml`` または ``edgechromium`` である。
    gui: GUIType | None = None,
    # debug： デバッグモードを有効にする。
    debug: bool = False,
    # http_server： 組み込みの HTTP サーバーを有効にする。有効にすると、ローカル・ファイルはランダムなポートでローカルのHTTPサーバーを使用して提供されます。
    # 各ウィンドウごとに、別々のHTTPサーバーが生成されます。このオプションは無視されます。
    http_server: bool = False,
    http_port: int | None = None,
    # user_agent： ユーザーエージェント文字列を変更します。
    user_agent: str | None = None,
    # private_mode： プライベート・モードを有効にする。プライベートモードでは、クッキーとローカルストレージは保存されません。
    private_mode: bool = True,
    # storage_path： クッキーと他のウェブサイトデータの保存場所。
    storage_path: str | None = None,
    # メニュー： アプリのメニューに含まれるメニューのリスト
    menu: list[Menu] = [],
    # server： サーバークラス。デフォルトはBottleServer
    server: type[http.ServerType] = http.BottleServer,
    # server_args： サーバーのインスタンス化に渡す引数の辞書。
    server_args: dict[Any, Any] = {},
    # ssl: ローカルHTTPサーバーのSSLを有効にします。デフォルトはFalse。
    ssl: bool = False,
    # icon: アイコンファイルへのパス： アイコンファイルへのパス。GTK/QTでのみサポート。
    icon: str | None = None,
):
```

DOM操作のサンプル  
pywebviewではJavaScriptを利用することなくDOM操作ができる。  
ボタンをクリックして、メッセージを表示するプログラムは次のようになる。  
```py
import webview
# HTMLを定義
HTML = """
<!DOCTYPE html><meta charset="UTF-8">
<style> div, button { font-size:3em; } </style>
<button id="btn">格言を表示</button>
<div id="msg"></div>
"""

# 最初に実行する関数)
def bind(window):
    window.dom.get_elements("#btn")[0].on("click", on_click)
# ボタンをクリックした時に実行する関数
def on_click(e):
    msg = window.dom.get_elements("#msg")[0]
    msg.text = "穏やかな舌は命の木である"
# Windowを作成
window = webview.create_window("click_test", html=HTML)
webview.start(bind, window)
```


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



<a id="Pyinstallerを使ったスクリプトのexe化"></a>
## Pyinstallerを使ったスクリプトのexe化

### exe化したいスクリプトがあるディレクトリで以下のコマンドを実行。  
`pyinstaller hoge.py --noconsole --onefile --icon=fuga.ico`

### 主なオプション
* `--noconsole` ： コンソール（コマンドプロンプト）を表示しない。
* `--onefile` ： 関連ファイルを1つにまとめてexeファイルを作成する。
* `--icon=../icon/fuga.ico`： exeファイルのアイコンを変更。*.icoファイルまでのパスを指定。
* `--name=SimpleTube`： exeファイルの名前を指定できる。




<a id="pipコマンド"></a>
## pipコマンド


### pip installコマンド
`pip install pandas` : 最新バージョンをインストール  
`pip install pandas==2.1.4` : 指定したバージョンをインストール  
`pip install --upgrade pandas` : 最新バージョンにアップグレード  
`pip install -r requirements.txt` : requirements.txtファイルに記述されているパッケージをインストール  
`pip install pyperclip pywebview pyinstaller` : 複数のモジュールを一括でインストール  

### pip uninstallコマンド
`pip uninstall pandas` : アンインストール  
`pip uninstall --yes pandas` : 確認プロンプトを表示せずにアンインストール  
`pip uninstall -r requirements.txt` : requirements.txtファイルに記述されているパッケージをアンインストール    

### パッケージを一覧するコマンド
`pip list` : インストールされているパッケージの一覧を出力  
`pip freeze` : インストールされているパッケージの一覧をrequirements.txtのフォーマットで出力  
`pip freeze > requirements.txt` : requirements.txtを作成  
`pip show pandas` : パッケージに関する情報を出力  


<a id="Flitを使ったPyPIへのパッケージ公開"></a>
## Flitを使ったPyPIへのパッケージ公開

### pyproject.tomlファイルを作成
Pythonプロジェクトのビルドや依存関係の管理に使用される設定ファイル。  
例：  
```pyproject.toml
[build-system]
requires = ["flit_core>=3.4"]
build-backend = "flit_core.buildapi"

[project]
name = "quickdriver"
version = "1.0.0"
dependencies = [
    "selenium >= 4.27.1",
    "pandas >= 2.2.3",
    "tqdm >= 4.67.1",
    "pyarrow >= 16.1.0",
]
requires-python = ">=3.8"
authors = [
    {name = "nishizawatakamasa"}
]
description = "A wrapper for Selenium WebDriver that simplifies browser automation."
readme = "README.md"
license = "MIT"
keywords = [
    "selenium"
]
classifiers = [
    "Programming Language :: Python :: 3",
    "Operating System :: OS Independent"
]

[project.urls]
repository = "https://github.com/nishizawatakamasa/quickdriver"
```

### ~/.pypircファイルを作成
設定を書き込んでおけば、 Flitでパッケージをアップロードする際にURL・ユーザ名・パスワードなどの入力を省くことができる。
```.pypirc
[distutils]
index-servers =
  pypi
  testpypi

[pypi]
username: __token__
password: <PyPI API token>

[testpypi]
repository = https://test.pypi.org/legacy/
username: __token__
password: <PyPI API token>
```

### venvで仮想環境を作成
ディレクトリ構成の例
```
┣ venv
┣ foo（パッケージ名）
┃    ┣ __init__.py
┃    ┗ foo.py
┣ .gitignore
┣ LICENSE
┣ pyproject.toml
┗ README.md
```

### 仮想環境内でコマンドを実行
Flitをインストール。  
`pip install flit`

コードをTestPyPIにアップロード。  
`flit publish --repository testpypi`

コードをPyPIにアップロード。  
`flit publish`


### バージョン更新のやりかた
1. pyproject.tomlのバージョンを変更
1. distディレクトリがある場合は削除
1. アップロード手順を実行※バージョンの変更がないと失敗する


[Flitドキュメント](https://flit.pypa.io/en/latest/)


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