# pandas覚書

* 目次
    * [基本的なこと](#基本的なこと)
    * [データへのアクセス](#データへのアクセス)
    * [文字列操作](#文字列操作)
    * [算術演算](#算術演算)
    * [数値操作](#数値操作)
    * [基礎集計](#基礎集計)
    * [ユニーク](#ユニーク)
    * [グルーピング](#グルーピング)
    * [横方向に結合](#横方向に結合)
    * [縦方向に結合](#縦方向に結合)
    * [任意の関数を適用](#任意の関数を適用)
    * [型変換](#型変換)
    * [欠損値の取り扱い](#欠損値の取り扱い)
    * [データ形式変換](#データ形式変換)
    * [melt](#melt)
    * [pivot](#pivot)
    * [pivot_table](#pivot_table)
    * [crosstab](#crosstab)
    * [その他](#その他)

<a id="基本的なこと"></a>
## 基本的なこと

### pandasの思想
1. DataFrameは単一のテーブル構造（行と列の形式）で扱う。  
1. 分離、適用、結合。

**思想に沿わない使い方をするべきではない。**

### DataFrameの基本
```py
# コンストラクタ
pandas.DataFrame(data=None, columns=None)


# DataFrameの各行を要素とするアプローチ
data = [
    {'col_0': 0, 'col_1': 1, 'col_2': 2},
    {'col_0': 3, 'col_1': 4, 'col_2': 5},
    {'col_0': 6, 'col_1': 7, 'col_2': 8},
]
# 辞書をDataFrameに変換
# 各要素(辞書)のキーが列名となり、値はそのまま値となる。
df = pd.DataFrame(data)
# DataFrameを辞書に変換
# 行名の情報は失われる。
di = df.to_dict(orient='records')

# DataFrameの各列を要素とするアプローチ
data = {
    'col_0': [0, 3, 6],
    'col_1': [1, 4, 7],
    'col_2': [2, 5, 8],
}
# 辞書をDataFrameに変換
# 各キーが列名となり、値が列になる。
# ※DataFrame作成時、各要素にリスト、タプル、rangeオブジェクト、シリーズを指定できる。
df = pd.DataFrame(data)
# DataFrameを辞書に変換
# 行名の情報は失われる。
di = df.to_dict(orient='list')

# DataFrameの各行を要素とするアプローチ(リスト)
data = [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
columns=['col_0', 'col_1', 'col_2']
# 二次元リストからDataFrameを作成。
# 各要素が行となる。
# columnsを省略すると、列名は0からの連番になる。
# ※DataFrame作成時、各要素にリスト、タプル、rangeオブジェクト、シリーズを指定できる。
df = pd.DataFrame(data, columns=columns)
# DataFrameを二次元リストに変換
# ndarrayのtolist()メソッドを使う。
# 要素は各行をリスト化したもの。
# df.values.tolist()でも可能だが非推奨
li = df.to_numpy().tolist()


DataFrame.to_numpy() # データ値属性(NumPy配列=ndarray)。
DataFrame.index = ['行名1', '行名2', '行名3'] # 行名属性へ値を代入。
DataFrame.columns = ['列名1', '列名2', '列名3'] # 列名属性へ値を代入。
DataFrame.index.to_list() # 行名属性をリスト化。
DataFrame.columns.to_list() # 列名属性をリスト化。
DataFrame.index.astype(str) # 行名属性を型変換。
DataFrame.columns.astype(str) # 列名属性を型変換。
DataFrame.index.name = 'indexタイトル' # index自体にタイトルをつける。※Noneの場合は消す。
DataFrame.columns.name = None # columns自体にタイトルをつける。※Noneの場合は消す。
```

### Seriesの基本
|||
|-|-|
|pd.Series(['hoge', 'fuga', 'piyo'])|リストから作成。インデックスは0からの連番となる。|
|pd.Series({'a': 1, 'b': 2, 'c': 3, 'd': 4})|辞書から作成。キーがインデックスとなる。|
|Series.to_frame('列名')|SeriesをDataFrameに変換。<br>引数で列名を指定できる(省略するとintの0になる)。<br>indexはそのまま。|
|Series.to_numpy()|データ値属性(NumPy配列=ndarray)。|
|Series.to_list()|シリーズをリストに変換。|
|Series.to_dict()|シリーズを辞書に変換。|


### データ型
pandasにはデータ型(dtype)が存在するが、int,str,floatのようなPythonの型を指定することもできる。  
この場合、等価なdtypeに自動的に変換される(Python3、64ビット環境での例は以下の通り)。
|Pythonの型|等価なdtypeの例|
|-|-|
|bool|bool|
|int|int64|
|float|float64|
|str|object（各要素がstr型）|

※注意点：int64は欠損値をサポートしていないため、数値型の列に欠損値が一つでも含まれていると自動的にfloat64へ変換される。

|他のdtype||
|-|-|
|'string'|文字列型。|
|datetime64[ns]|日付、時刻型。[ns]はナノ秒(nanoseconds)単位の精度を示す。|
|'category'|※このデータ型は完全にはサポートされていないため、基本的には使わないようにする。<br>性別などのカテゴリを数値化し、0や1で示すようなデータ。<br>つまり、数値自体に意味のない数値データ。<br>また、文字列でカテゴリ分けしてあるデータ。|

※注意点：文字列や日付、時刻などは、デフォルトでは柔軟なobject型が設定される。


<a id="データへのアクセス"></a>
## データへのアクセス

### 基本

df.at['行名', '列名']  

df.loc[  
boolのSeries、行名、行名リスト、行名スライス,  
boolのSeries、列名、列名リスト、列名スライス  
]


* atプロパティを使ったアクセス
    * 行名と列名を指定してスカラーにアクセスする。
* locプロパティを使ったアクセス
    * 行名と列名を指定するとスカラーにアクセスするが、atを使用するべき(処理速度も速い)。
    * どちらか片方が行名や列名の場合はSeriesにアクセスする。
    * それ以外の場合はDataFrameにアクセスする。
    * ※行名や列名がint型の場合は、指定もint型で。
    * ※デフォルトの連番index値の行名はint型。
    * ※行や列をリストで指定した場合はその順番でのアクセスとなる。
    * boolのSeriesを指定すると、Trueの行や列が選択される(Boolean Indexing)。
        * 行選択と列選択どちらの場合でもindexが一致している必要がある。
        * 複数のboolのSeriesに~&|を適用し、複数条件で選択することも可能。
        * 優先順位が高い順から、~(not)、&(and)、|(or)
        * 比較演算子を使うときは括弧で括る。
        * 優先したい処理も括弧で括る。
* 操作
    * 取得
        * アクセスしたデータをそのまま戻り値として取得できる。
    * 更新、追加
        * アクセスしたデータに新しいデータを代入する。
        * 存在する単一セル、行、列に対してはデータ更新となる。
        * 存在しない単一セル、行、列に対してはデータ追加となる。


### データの代入
* 単一セルに対して
    * スカラーを代入
        * そのまま代入するだけ。
* 行に対して
    * スカラーを代入
        * そのまま代入するだけ。
    * Seriesを代入
        * 完全な行単位で行う。
        * サイズ(行数、列数)とデータ型を合わせる。
        * indexを合わせて代入される。
        * 行Seriesでは列名がindexとなる。
* 列に対して
    * スカラーを代入
        * そのまま代入するだけ。
    * Series、DataFrameを代入
        * 完全な列単位で行う。
        * サイズ(行数、列数)とデータ型を合わせる。
        * indexを合わせて代入される。
        * 1列を指定する場合、基本はSeriesにアクセスする。


### インデックス指定による簡潔な書き方
|||
|-|-|
|df[boolのSeries]|行の選択。df.loc[boolのSeries]と同じ。|
|df['行名1':'行名2']|行の選択。行名のスライスで該当した行をDataFrameとして取得。<br>※指定できる行名は文字列のみ。つまり、intの行名を指定することは不可。|
|df[列名]|列の選択。df.loc[:, 列名]と同じ。|
|df[列名リスト]|列の選択。df.loc[:, 列名リスト]と同じ。|
|df[boolのDataFrame]|true部分の値のみを抽出。他は欠損値となる。<br>※サイズが一致していなくてもエラーは出ないが、想定されている使い方ではない気がする。|

### 正規表現による行、列の選択
|||
|-|-|
|df.filter(regex=r'3')|正規表現の条件を満たす列名の列を選択。<br>DataFrameを返す。<br>axis=0にすると行に適用。<br>行と列を同時に指定することはできない。|

### Seriesに対するインデックス指定
|||
|-|-|
|s[ラベル名]|単独の要素の値をそれぞれの型で取得|
|s[ラベル名のリスト]|単独・複数の要素の値をpandas.Seriesとして取得|
|s[ラベル名・番号のスライス]|単独・複数の要素の値をpandas.Seriesとして取得。<br>intのスライスは、常にラベル番号のスライスとして扱われる。つまりintのラベル名を指定することは不可。|
|s[boolのSeries]|Trueの要素をpandas.Seriesとして取得|

### boolのSeriesを得る方法の例
|||
|-|-|
|Series >= 数値||
|Series == '文字列'||
|Series != '文字列'||
|Series1 >= Series2||
|Series.isin(['文字列1', '文字列2'])<br>Series.isin([数値1, 数値2])|指定した複数の文字列のいずれかと完全一致。<br>指定した複数の数値のいずれかと完全一致。|
|★Series.str.contains(pat)|指定した正規表現が文字列内に含まれているかどうかで判定。<br>引数naで欠損値の判定を指定(TrueかFalse、デフォルトはNone)。|
|Series.isnull()||
|Series.notnull()||
|DataFrame.duplicated()<br>Series.duplicated()|重複行を抽出(True)<br>引数subsetで重複判定する列を指定(デフォルトでは全ての列)<br>subset='列名'<br>subset=['列名1', '列名2']<br>引数keep(デフォルトでは、重複した最初の行は重複行として扱わない)<br>keep='last'とすると、重複した最後の行は重複行として扱わない。<br>keep=Falseとすると、重複した全ての行を重複行として扱う。|
|df_of_bool.all(axis=1)|※boolのDataFrameに対して使用<br>行が全てTrue|
|df_of_bool.any(axis=1)|※boolのDataFrameに対して使用<br>行がひとつでもTrue|

### boolのDataFrameを得る方法の例
|||
|-|-|
|DataFrame >= 数値||
|DataFrame == '文字列'||
|DataFrame != '文字列'||
|df1 == df2|※比較できるのはDataFrameのサイズが一致している場合のみ。|
|df1 >= df2|※比較できるのはDataFrameのサイズが一致している場合のみ。|
|DataFrame.isin(['文字列1', '文字列2'])|指定した複数の文字列のいずれかと完全一致。|
|DataFrame.isnull()||
|DataFrame.notnull()||



### 基本(iat,iloc)

df.iat['行番号', '列番号']  

df.iloc[  
行番号、行番号リスト、行番号スライス,  
列番号、列番号リスト、列番号スライス  
]

### インデックス指定による簡潔な書き方(iat,iloc)
|||
|-|-|
|df[2:9]|行の選択。行番号のスライスで該当した行をDataFrameとして取得。|



<a id="文字列操作"></a>
## 文字列操作

### 基本
|||
|-|-|
|df['str1'] + ' in ' + df['str2']||
|Series.str[2]||
|Series.str[2:9]||
|Series.str.strip()||
|Series.str.join(sep)||
|Series.str.normalize('NFKC')||
|Series.str.zfill(6)|左側をゼロ埋めし、指定した文字数にする|

### 正規表現関連
|||
|-|-|
|★Series.str.extract(pat)|最初のマッチ部分のみ抽出。<br>キャプチャ部分がそれぞれ列となったDataFrameを返す。 <br>キャプチャに名前を付けると、それがそのまま列名となる。|
|Series.str.extractall(pat)|すべてのマッチ部分を抽出。<br>キャプチャ部分がそれぞれ列となり、マッチ部分がそれぞれ行となったマルチインデックスのDataFrameを返す。<br>※マッチする部分が一つしかなくてもindexはマルチインデックスとなる。<br>キャプチャに名前を付けると、それがそのまま列名となる。|
|Series.str.findall(pat)||
|Series.str.replace(pat, repl, regex=True)|※ DataFrame.replace, Series.replaceのほうが便利。|
|Series.str.split(pat, regex=True)|patが初期値のままなら空白で分割される。<br>expand=Trueとすると複数の列に分割。|

|||
|-|-|
|★DataFrame.replace({<br>'列名1': {'pat1': 'repl1', 'pat2': 'repl2'},<br>'列名2': {'pat3': 'repl3'},<br>})<br><br>★DataFrame.replace({'pat1': 'repl1', 'pat2': 'repl2'})<br>★Series.replace({'pat1': 'repl1', 'pat2': 'repl2'})|セル値の置換。文字列と数値に対して使える。<br>引数regexの初期値はFalse。Trueとすると正規表現での置換になる。<br>キャプチャグループを設定した場合、グループにマッチした文字列を1つ目から\1, \2, \3...とrepl内で使用できる。<br>replも基本的にraw文字列を使うのがベター。|


#### インラインフラグ
|||
|-|-|
|r'(?i)hoge'|re.IGNORECASE<br>re.I|
|r'(?m)hoge'|re.MULTILINE<br>re.M|
|r'(?s)hoge'|re.DOTALL<br>re.S|
|r'hoge(?s:fuga)piyo'|部分適用|
|(?im)|複数フラグ|
|(?ms:)|複数フラグを部分適用|


<a id="算術演算"></a>
## 算術演算

DataFrame、Series、スカラー値は、それぞれ算術演算子(+、-、*、/、//、%、**)による処理が可能。  
式は括弧()で一括りにできる。

※DataFrameとSeriesの算術演算は、SeriesをDataFrameの各行に対して適用する挙動になる。

累算代入演算子(+=、-=、*=、/=、//=、%=、**=)も使える。


<a id="数値操作"></a>
## 数値操作

### 基本的な操作
|||
|-|-|
|df['num1'] * 3 / df['num2']||
|df[['num1', 'num2']] /= 3||
|(df['num'] / 30).astype(str).str[0:3].astype(float)||
|(df['num'] / 30).astype(int)||


<a id="基礎集計"></a>
## 基礎集計

### 重要なメソッド
|||
|-|-|
|sum()|合計を算出。|
|min()|最小値を算出。|
|max()|最大値を算出。|
|mean()|平均値を算出。|
|median()|中央値を算出。|
|std()|標準偏差を算出。|
|count()|非欠損値数を算出。|
|describe()|主要な統計量を一括算出。|
|nunique()|ユニーク数を算出。|

#### DataFrameから呼び出す場合
* Seriesを返す。
* デフォルトでは列方向への計算。axis=1とすると行方向への計算。

#### Seriesから呼び出す場合
* スカラー値を返す。



<a id="ユニーク"></a>
## ユニーク

### 基本
|||
|-|-|
|Series.unique()|ユニークな要素の値の一覧を一次元のNumPy配列=ndarrayで返す。<br>欠損値NaNも含まれる。<br>出現順に並べられる。<br>ndarrayでなければならない理由がないのなら、tolist()メソッドでリストに変換して使う。|
|Series.value_counts()|ユニークな要素の値をインデックス、その個数を要素とするSeriesを返す。<br>dropna=FalseとするとNaNもカウント対象となる。<br>normalize=Trueとすると、合計が1になるように規格化した値となる。|
|DataFrame.value_counts(subset=['列名1', '列名2'])|基本はSeries.value_counts()と同じだが、subsetで指定した列要素の組み合わせ単位でユニークカウントされる。<br>subsetを指定しなければ、全ての列要素の組み合わせ単位になる。|


<a id="グルーピング"></a>
## グルーピング

### 基本的なグルーピング
|||
|-|-|
|dfg = DataFrame.groupby('列名')|列名で指定した列の値ごとにグルーピングされる。<br>DataFrameGroupByオブジェクトを返す。<br>デフォルトでは指定した列が結果のインデックスになるが、as_index=Falseとするとインデックスにならない。<br>デフォルトでは指定した列に含まれる欠損値NaNは無視されるが、dropna=FalseとするとNaNも一つのキーとして扱われる。|

### DataFrameGroupByオブジェクトとは
指定列の値ごとに分割された複数のDataFrame、というイメージ。  
当然、Boolean Indexingでも特定の列の値を指定してDataFrameを取得することはできる。  
特定のカテゴリーだけ選択したいのならBoolean Indexingを使い、全カテゴリーを分割し選択できるようにしたいのならgroupbyメソッドを使うといい。  
ちなみにイテラブルオブジェクトである。

```py
# for文を使うと、グループ名とそのグループのDataFrameを順に取得できる。
for group, df in dfg:
    print(group, df.shape)
```

### DataFrameGroupByオブジェクトの主なメソッド

#### 指定したグループのDataFrameを取得
|||
|-|-|
|dfg.get_group('グループ名')|グループ名で指定したグループのDataFrameを取得できる。|

#### グループごとにデータを集約(各DataFrameに適用し、全て結合するイメージ)
|||
|-|-|
|dfg.size()|行数|
|dfg.sum()|合計|
|dfg.mean()|平均|
|dfg.max()|最大|
|dfg.min()|最小|
|dfg.median()|中央値|
|dfg.std()|標準偏差|
|dfg.count()|欠損値ではない要素数|
|dfg.describe()|主要な統計量を一括算出|
|dfg.agg(func)|任意の関数適用<br>※自作関数(def or lambda)を指定する場合は、引数がarray-likeとなるように作成。 |

### SeriesGroupByオブジェクト
DataFrameGroupByオブジェクトに対して列名を指定すると取得できる。  
sg = dfg['列名']  
分割された複数のDataFrameが各々Seriesになる、というイメージ。  
DataFrameGroupByオブジェクトと同じように集約メソッドなどが使える。

### 参考サイト
* [pandasのgroupby()でグルーピングし統計量を算出](https://note.nkmk.me/python-pandas-groupby-statistics/)  
* [pandas groupbyでグループ化｜図解でわかりやすく解説](https://www.yutaka-note.com/entry/pandas_groupby)


<a id="横方向に結合"></a>
## 横方向に結合

### 基本
```py
# マージ関数
df = pd.merge(df_left, df_right, on='キーとする列の名前', how='left')
```

### 結合処理の基本的な挙動。
共通するキーを持つ行同士の全組み合わせが生成される。

### 引数on、left_on、right_on、left_index、right_index
キーにする列やインデックスを指定

* デフォルトでは、すべての同名の列がキーとして処理される。
* キーとする列を明示的に指定する場合は、引数on, left_on, right_onに列名を指定する(複数の場合はリスト)。
* ※キーとする列を省略して問題ない場合でも、明示しておいたほうが分かりやすい。
* onは両側dfの列名, left_onはleft側dfの列名, right_onはright側dfの列名。
* 複数の列をキーとする場合、キーに指定した全ての列の値が一致することが結合の条件となる(複合キー)。
* indexをキーに指定する場合は、left_index、right_indexをTrueとする(left_on、right_onと組み合わせることも可能。)。

### 引数suffixes
任意のサフィックスを指定

キーにする列以外の列名が重複している場合、デフォルトでは_x(left側), _y(right側)というサフィックスがつけられる。  
suffixes=[left用サフィックス, right用サフィックス]とすると、任意のサフィックスを指定できる。  
例：suffixes=['_left', '_right']  

### 引数how
結合方法を指定

|||
|-|-|
|how='inner'|leftとright両方に共通するキーを持つ行のみが残る。<br>これがデフォルト。|
|how='left'|innerに加えて、leftにしかキーが存在しない行も残る。|
|how='right'|innerに加えて、rightにしかキーが存在しない行も残る。|
|how='outer'|innerに加えて、leftにしかキーが存在しない行とrightにしかキーが存在しない行が残る。<br>(要するにleftとrightのすべての行が残る。)|

### 引数indicator
元データの情報を含む列が追加される。

indicator=Trueとすると'_merge'という名前の列が追加され、both, left_only, right_onlyのいずれかに分類される。

### 参考サイト
* [pandas.DataFrameを結合するmerge, join（列・インデックス基準）](https://note.nkmk.me/python-pandas-merge-join/)  


<a id="縦方向に結合"></a>
## 縦方向に結合

### 基本
```py
# コンキャット関数
# キーには列名を使用する。
df = pd.concat([df1, df2, df3])
```

### 引数join
結合方法を指定

|||
|-|-|
|join='outer'|すべての列が残る。<br>これがデフォルト。|
|join='inner'|共通の名前の列のみが残る。|


<a id="任意の関数を適用"></a>
## 任意の関数を適用

### map
|||
|-|-|
|Series.map(len)<br>Series.map(lambda x: func(x, 5))<br>DataFrame.map(len)<br>DataFrame.map(lambda x: hex(int(x)))|引数に指定した関数を各要素に適用する(新しいオブジェクトが返される)。<br>defで定義した関数やラムダ式も指定可能。<br>na_action='ignore'とすると、NaNは関数に渡されずに結果がそのままNaNとなる。|

### apply
|||
|-|-|
|※特殊ケース<br>Series.apply(pd.Series)|Seriesの各要素のリストにpd.Series()を適用したDataFrameを返す。<br>※Series.map()はDataFrameを返すことができないため、Series.apply()を使う。|
|DataFrame.apply(lambda s: s[1] in s['所在地'], axis=1)|第一引数に指定した関数をDataFrameの各行もしくは各列に一括で適用する。<br>新しいオブジェクト(SeriesかDataFrame)が返される。<br>defで定義した関数やラムダ式も指定可能。<br>デフォルトでは各列がSeriesとして関数に渡される(戻り値が新しい列となる)。<br>引数axisを1とすると各行がSeriesとして関数に渡される(戻り値が新しい行となる)。<br>Laxis(0と1の覚え方)。<br>Seriesを引数として受け取れない関数だとエラーになる。|

### 注意点
mapやapplyの処理速度は遅い。あくまでも他では実現できない複雑な処理を適用するためのもの。


<a id="型変換"></a>
## 型変換

|||
|-|-|
|s2 = pd.to_numeric(s1, errors='coerce')|シリーズを数値型に変換する(文字列等の例外データは欠損値に変換する)。|
|DataFrame.astype(int)<br>Series.astype(float)|データ型を一括で変更する。<br>DataFrame.astypeの場合は引数に{'列名': 'string'}のような辞書も指定でき、任意の列のデータ型を個別に変更できる。<br>※変換できない値が含まれている場合はValueErrorとなる。|

```py
# datetime64[ns]型への変換は特殊で、to_datetime関数を使う。
df['date'] = pd.to_datetime(df['date'])

# to_datetimeで変換できる主な文字列
'2023-02-28'
'2023/02/28'
'20230228 14:21:31'

# 標準的な書式でない場合は、引数formatに書式文字列を指定する。
df['date'] = pd.to_datetime(df['date'], format='%Y年%m月%d日')
```
※引数formatに指定する書式化コードは[公式ドキュメント](https://docs.python.org/ja/3/library/datetime.html#strftime-and-strptime-behavior)を参照。


<a id="欠損値の取り扱い"></a>
## 欠損値の取り扱い

|||
|-|-|
|DataFrame.dropna(how='any', subset=None)|欠損値を含む行を削除する。<br>※ how='all'とすると、全てが欠損値の行を削除する。<br>※ subset=['列名1', '列名2']とすると、指定列のみが調査対象になる。|
|DataFrame.fillna(value)<br>DataFrame.fillna({'列名1': 'value1', '列名2': 'value2'})<br>DataFrame.fillna(Series)<br>Series.fillna(value)|欠損値をvalueで指定する値に置き換える。|
|DataFrame.ffill()<br>Series.ffill()|欠損値を直前の非欠損値で置き換える。|
|DataFrame.bfill()<br>Series.bfill()|欠損値を直後の非欠損値で置き換える。|


<a id="データ形式変換"></a>
## データ形式変換

### pd.read_parquet
```py
df = pd.read_parquet('hoge/fuga/piyo.parquet')
```
* ファイルパスからparquetオブジェクトを読み込み、DataFrameを返す。

### pd.read_csv 
|主な引数|引数の説明|
|-|-|
|filepath_or_buffer|CSVファイルのパスを指定。唯一の必須引数。位置引数でもある。|
|sep=','|区切り文字を指定。デフォルトはカンマ。|
|header=None|列名(columns)の設定。デフォルトでは一行目。Noneにすると0始まりの連番。|
|usecols=[2, 6, 7, 8]|読み込む列を、列名のリストで指定(読み込む列を限定するため、メモリの節約にもなる)。|
|dtype=str|カラムのデータ型を指定。<br>引数に{'列名': str}のような辞書も指定でき、任意の列のデータ型を個別に指定できる。|
|encoding|文字コードを指定。デフォルトはNoneだが、内部の処理的には"utf-8"と同じ。<br>UnicodeDecodeErrorが発生した場合に試す選択肢→"shift-jis"、"cp932"、"utf-8-sig"。|
|index_col|index_col='date'のように、indexとするカラムを列名(または列番号)で指定できる。省略すると新しいindex(0からの連番)が作成される。|
|nrows=2|読み込む行数（ヘッダー行数を除く）を指定。中身をざっと確認したい時用。|
|on_bad_lines|フィールド数がヘッダーのカラム数よりも多いレコード（不正レコード）の扱いを指定。<br>デフォルトはerrorで、ParserErrorが発生する。|

### pd.read_clipboard
* 内部的にはpd.read_csvが動いているため、基本的な引数や挙動はpd.read_csvと同じ。
* 違い
    * データはファイルからではなくクリップボードから読み込まれる。
    * 引数sepのデフォルト値が空白文字(sep=r'\s+')。
```py
# 表データの左上隅が空の場合、pandasがデータ構造を誤って解釈する可能性がある。
# その場合に使う処理の一例。
pyperclip.copy('Placeholder' + pyperclip.paste())
df = pd.read_clipboard('\t', header=None)
df.iat[0, 0] = None
```

### pd.read_excel
`$ pip install openpyxl`

|主な引数|引数の説明|
|-|-|
|io|Excelファイルのパスを指定。|
|sheet_name|読み込むシートを指定。0始まりの番号かシート名で指定。デフォルトは0。|
|header=None|pd.read_csvと同様。|
|usecols=[2, 6, 7, 8]|読み込む列をリストで指定する。intのリストの場合は列番号（0始まり）での指定となり、文字列のリストの場合は列名での指定となる。列名がintの場合、その列名を指定することはできない。|
|dtype=str|pd.read_csvと同様。|
|index_col|indexとするカラムを列番号で指定できる。省略すると新しいindex(0からの連番)が作成される。|

### pd.read_html
`$ pip install lxml html5lib beautifulsoup4`
```py
dfs = pd.read_html(url, match='リリース日', header=0)
```
* 第一引数にURLかHTMLファイルへのパスを指定すると、そのページ内のtable表をすべて取得しDataFrameのリストとして返す。
* 表が1つしかない場合も、長さ1のリストとして返される。
* 引数のmatchの文字列を指定すると、その文字列が含まれる表のみを取得できる。
* 引数のheaderで指定した行がヘッダーになる。

---------------------------------------------------

### df.to_parquet
`$ pip install pyarrow`
```py
df.to_parquet('hoge/fuga/piyo.parquet')
```
* .parquetファイルとして新規作成or上書き保存。

### df.to_csv
```py
df.to_csv('hoge/fuga/piyo.csv',  sep=',', header=True, index=True, encoding=utf-8, mode='w')
```
* 引数のheaderとindexは、それぞれの出力の有無をboolで指定する。
* .csvファイルとして出力。

### df.to_clipboard
```py
df.to_clipboard(excel=True, sep=r'\t', **kwargs)
```
* 内部的にはto_csvが動いているため、基本的な引数や挙動はto_csvと同じ。
* 違い
    * データはファイルとして出力されず、クリップボードにコピーされる。
    * 引数sepのデフォルト値がタブ(sep=r'\t')
    * excel=Falseとすると、print(df)で表示される文字列がそのままクリップボードに書き込まれる。

### df.to_excel
`$ pip install openpyxl`
```py
df.to_excel('hoge/fuga/piyo.xlsx', sheet_name='Sheet1', header=True, index=True)
```
* 引数のheaderとindexは、それぞれの出力の有無をboolで指定する。
* .xlsxファイルとして新規作成or上書き保存。

### df.to_html
```py
df.to_html('hoge/fuga/piyo.html', header=True, index=True)
```
* 引数のheaderとindexは、それぞれの出力の有無をboolで指定する。
* .htmlファイルとして新規作成or上書き保存。
* ファイルパスを指定しなければ、HTMLのテキストを返す。

### df.to_markdown
`$ pip install tabulate`
```py
df.to_markdown('hoge/fuga/piyo.md', index=True, mode='w')
```
* .mdファイルとして出力。
* ファイルパスを指定しなければ、Markdownのテキストを返す。


### 参考サイト
* [【保存版】Pandas2.0のread_csv関数の全引数、パフォーマンス、活用テクニックを完全解説する！](https://qiita.com/fujine/items/dbe2f5e4101d6299ff12#encoding)


<a id="melt"></a>
## melt

### 基本
横持ちのDataFrameを縦持ちのDataFrameに再構築する(DB的になる)。  
コンピュータにとってわかりやすい表になるイメージ。
```py
# DataFrameを三種類のカラム(id_vars, variable, value)に再構築して返す。
df_melted = df.melt(
    # 列名か列名リストを指定。指定した列はid_varsカラムとなり、meltされずにそのまま残る。
    id_vars,
    # 列名か列名リストを指定。指定した列の列名はvariableカラムとなり、値はvalueカラムとなる。
    # デフォルトでは、id_varsとして設定されていないすべての列。
    value_vars,
    # variableカラムの列名を指定(デフォルトはcolumns自体のタイトルで、無い場合は'variable')。
    var_name,
    # valueカラムの列名を指定(デフォルトは'value')。
    value_name,
)
```

### 再構築したDataFrameのイメージ
|id_vars|variable(列名はvar_name)|value(列名はvalue_name)|
|-|-|-|
|id_vars指定列(全列、全行)|value_vars指定列の列名(1列目)|value_vars指定列(1列目、全行)|
|id_vars指定列(全列、全行)|value_vars指定列の列名(2列目)|value_vars指定列(2列目、全行)|
|id_vars指定列(全列、全行)|value_vars指定列の列名(3列目)|value_vars指定列(3列目、全行)|
|id_vars指定列(全列、全行)|value_vars指定列の列名(4列目)|value_vars指定列(4列目、全行)|
|id_vars指定列(全列、全行)|value_vars指定列の列名(5列目)|value_vars指定列(5列目、全行)|


### 参考サイト
* [pandas.melt — pandas 2.2.2 documentation](https://pandas.pydata.org/docs/reference/api/pandas.melt.html)
* [データフレームを再構築するPandasのMelt()関数のお話し](https://www.salesanalytics.co.jp/datascience/datascience021/)



<a id="pivot"></a>
## pivot

重複するエントリがなく(あったらエラーが発生)、単にmeltの逆処理をしたい場合に使う。
指定したindexとcolumnsの組み合わせの対象データが存在しない場合、その項目はNaNになる。    
人間にとってわかりやすい表になるイメージ。


```py
# DataFrameの三種類のカラム(id_vars, variable, value)を橫持ちに再構築して返す。
df_pivoted = df.pivot(
    # id_varsに対応する列名リストを指定。
    index,
    # variableに対応する列名を指定。
    columns,
    # valueに対応する列名を指定。
    values,
).reset_index() # indexを振り直す。
# columnsのタイトルを消す。
df_pivoted.columns.name = None
```


<a id="pivot_table"></a>
## pivot_table

重複するエントリを集計したい場合に使う。
指定したindexとcolumnsの組み合わせの集計対象データが存在しない場合、その項目の集計結果はNaNになる。
使用前に欠損値の処理を済ませておくのが無難。  


```py
df_pt = df.pivot_table(
    # 必須。列名か列名リストを指定。結果の行見出し。
    index,
    # 必須。列名か列名リストを指定。結果の列見出し。
    columns,
    # 列名か列名リストを指定。集計対象のデータ。デフォルトはindexとcolumnsに指定していないデータ型が数値の列全て。
    values,
    # 集計方法を文字列で指定するか、自分で集計関数を定義して指定する。
    # 自作の集計関数は、引数でSeriesを受け取り、スカラー値を返す必要がある。
    # 集計方法と自作の集計関数は、リストとして複数指定することも可能。
    # 以下は文字列で指定できる集計方法
    # 'mean': 平均値。デフォルト。
    # 'median': 中央値。
    # 'sum': 合計。
    # 'prod': 積。
    # 'min': 最小値。
    # 'max': 最大値。
    # 'count': 要素の個数をカウント。pd.crosstab()と同様に、カテゴリごとの出現回数が算出できる。
    # 'std': 標準偏差。
    # 'var': 分散。
    # 'first': 最初の値を取得。
    # 'last': 最後の値を取得。
    aggfunc,

    # Trueとすると、小計と総計も算出できる。
    margins: bool = False,
    # 小計・総計の行ラベル・列ラベルを文字列で指定。デフォルトは'All'。
    margins_name: Level = "All",

    # 結果をソートするかどうかを指定。
    sort: bool = True,
    # 保留
    observed: bool | lib.NoDefault = lib.no_default,
)
```


<a id="crosstab"></a>
## crosstab

### 基本
```py
# クロス集計表を作成する関数
# crosstabで出来ることは基本的にpivot_tableでも出来るので、pivot_tableを使う。
# ただし、crosstabだと結果を1に規格化（正規化）したりできる。
df = pd.crosstab()
```

```py
def crosstab(
    # 行に表示するカテゴリ変数(Series、またはSeriesのリスト)
    # Seriesのリストを指定した場合、Multiindexになる。
    index,
    # 列に表示するカテゴリ変数(Series、またはSeriesのリスト)
    # Seriesのリストを指定した場合、Multiindexになる。
    columns,
    # index名をリストで指定。
    # 要素数が、indexに指定したSeries数と一致する必要がある。
    rownames=None,
    # columns名をリストで指定。
    # 要素数が、columnsに指定したSeries数と一致する必要がある。
    colnames=None,
    # Trueで集計表に小計を追加する。
    margins: bool = False,
    # 小計の項目名を文字列で指定。
    margins_name: Hashable = "All",
    # aggfuncで集計する量的変数(Series)。
    # aggfuncを同時に指定する必要がある。
    values=None,
    # valuesを集計する方法を文字列で指定(主に'sum'、'max'、'min'、'mean'、'median')。
    # valuesを同時に指定する必要がある。
    aggfunc=None,
    # True(初期値)だと、全てが欠損値の列を含めない。
    # Falseだと含める。
    dropna: bool = True,
    # 全ての値を値の合計で割って正規化する。
    # False(初期値)だと正規化されない。
    # 'all'が渡されると全ての値が正規化される。
    # 'index'が渡されると各行の値が正規化される。
    # 'columns'が渡されると各列の値が正規化される。
    # ※marginsがTrueの場合、小計値も正規化される。
    normalize: bool = False,
) -> DataFrame:
```



<a id="その他"></a>
## その他

### ランダムサンプリング
|||
|-|-|
|DataFrame.sample(n=30)|n行だけランダムサンプリング|
|DataFrame.sample(frac=0.1)|行を指定割合だけランダムサンプリング|


### その他
|||
|-|-|
|DataFrame.reset_index()<br>Series.reset_index()|indexを0からの連番に振り直す。<br>元のindexはデータ列として残る(元のindex名がそのまま列名となる。ない場合ば'index'となる)。<br>drop=Trueとすると、元のindexは削除され残らない。<br>※引数にignore_index=Trueを指定できるメソッドに対しても、こっち(reset_index)を使うほうがいいかも。|
|DataFrame.drop(index=['行名1', '行名2'], columns=['列名1', '列名2'])|DataFrameの行・列を指定して削除する。|
|DataFrame.drop_duplicates()<br>Series.drop_duplicates()|重複行を削除する。<br>※引数(subset、keep)による重複行の扱い方はduplicatedメソッドと同じ。|
|DataFrame.sort_values(by=['列名1', '列名2'], ascending=[True, False])|引数byで列名を指定してソートする。<br>引数ascendingではTrueが昇順でFalseが降順。<br>byとascendingでは、同じ位置にある要素がそれぞれ対応する。<br>na_position='first'とすると、欠損値NaNが先頭に並べられる。デフォルトでは末尾。|
|DataFrame.transpose()|転置|
|DataFrame.copy()|dfのコピーを作成(参照の値渡しを防ぐため)。|
|DataFrame.rename(index={'元の行名': '新しい行名'}, columns={'元の列名': '新しい列名'})|行名・列名のいずれかのみを変更したい場合は、引数indexとcolumnsのどちらか一方だけを指定すればよい。|
|Series.shift(2, fill_value='あいうえお')|行方向に値がずれた列を作成する。<br>第一引数にずらし幅を指定。fill_valueにずれた部分に設定する値を指定(省略すると欠損値)。|

### 保留
* ループ処理(iterrows、itertuples)
* 欠損値を前後の値から線形補間(interpolate)
* agg
* pandas.DataFrame.update()※DataFrameの値の更新
