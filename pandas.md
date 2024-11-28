# pandas覚書

* 目次
    * [基本的なこと](#基本的なこと)
    * [選択操作](#選択操作)
    * [文字列操作](#文字列操作)
    * [算術演算](#算術演算)
    * [基礎集計](#基礎集計)
    * [ユニーク](#ユニーク)
    * [グルーピング](#グルーピング)
    * [横方向に結合](#横方向に結合)
    * [縦方向に結合](#縦方向に結合)
    * [任意の関数を適用](#任意の関数を適用)
    * [型変換](#型変換)
    * [欠損値の取り扱い](#欠損値の取り扱い)
    * [データ形式変換](#データ形式変換)
    * [クロス集計](#クロス集計)
    * [メルト](#メルト)
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

# 二次元リストからDataFrameを作成。
# 各要素が行となる。
# columnsを省略すると、列名は0からの連番になる。
data = [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
columns=['col_0', 'col_1', 'col_2']
df1 = pd.DataFrame(data, columns=columns)
# DataFrameを二次元リストに変換
# ndarrayのtolist()メソッドを使う。
# 要素は各行をリスト化したもの。
# df1.values.tolist()でも可能だが非推奨
li = df1.to_numpy().tolist()
# [[0, 1, 2], [3, 4, 5], [6, 7, 8]]

# 辞書からDataFrameを作成
# 各キーが列名となり、値が列になる。
data = {
    'col_0': [0, 1, 2],
    'col_1': [3, 4, 5],
    'col_2': [6, 7, 8],
}
df2 = pd.DataFrame(data)
# DataFrameを辞書に変換
# 行名の情報は失われる。
di = df2.to_dict(orient='list')
# {
#     'col_0': [0, 1, 2],
#     'col_1': [3, 4, 5],
#     'col_2': [6, 7, 8],
# }

# 上記2パターンのDataFrame作成方法では、各要素にリスト、タプル、rangeオブジェクト、シリーズを指定できる。


DataFrame.to_numpy() # データ値属性(NumPy配列=ndarray)。
DataFrame.index = ['行名1', '行名2', '行名3'] # 行名属性へ値を代入。
DataFrame.columns = ['列名1', '列名2', '列名3'] # 列名属性へ値を代入。
DataFrame.index.to_list() # 行名属性をリスト化。
DataFrame.columns.to_list() # 列名属性をリスト化。
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



<a id="選択操作"></a>
## 選択操作

### 基本
|||
|-|-|
|df.loc[<br>boolのSeries、行名、行名リスト、行名スライス,<br>boolのSeries、列名、列名リスト、列名スライス<br>]|※ 行名や列名がint型の場合は、指定もint型で。※index値の行名はint型。<br>■行名と列名で指定すると単一要素の値にアクセスすることが可能だが、atを使用するべき(処理速度も速い)。<br>■どちらか片方が行名や列名の場合、選択した行や列をSeriesとして取得・変更・追加できる。<br>・ 変更、追加時にはスカラー値、Series(indexを合わせて代入されることに注意)、リスト、NumPy配列を代入。<br>■それ以外の場合は、選択した範囲をDataFrameとして取得・変更できる。<br>・ 変更時にはスカラー値、Series(indexを合わせて代入されることに注意)、二次元リスト、二次元NumPy配列を代入。<br>■行や列をリストで指定した場合はその順番で選択される。<br>■boolのSeriesを指定すると、Trueの行や列が選択される。<br>・Boolean Indexingという。<br>・行選択と列選択どちらの場合でもindexが一致している必要がある。<br>・ 複数のboolのSeriesに~&\|を適用し、複数条件で選択することも可能。<br>・ 優先順位が高い順から、~(not)、&(and)、\|(or)<br>・ 比較演算子を使うときは括弧で括る。<br>・ 優先したい処理も括弧で括る。|
|df.at['行名', '列名']|選択した単一要素の値を取得・変更できる。<br>変更時にはスカラー値を代入。|

### インデックス指定による簡潔な書き方
|||
|-|-|
|df[列名]|列の選択。df.loc[:, 列名]と同じ。|
|df[列名リスト]|列の選択。df.loc[:, 列名リスト]と同じ。|
|df[boolのSeries]|行の選択。df.loc[boolのSeries]と同じ。|
|df[boolのDataFrame]|true部分の値のみを抽出。他は欠損値となる。<br>※サイズが一致していなくてもエラーは出ないが、想定されている使い方ではない気がする。|


### 正規表現による行、列の選択
|||
|-|-|
|df.filter(regex=r'3')|正規表現の条件を満たす列名の列を選択。<br>DataFrameを返す。<br>axis=0にすると行に適用。<br>行と列を同時に指定することはできない。|


### Seriesに対するインデックス指定
|||
|-|-|
|s[ラベル名・番号]|単独の要素の値をそれぞれの型で取得|
|s[ラベル名・番号のリスト]|単独・複数の要素の値をpandas.Seriesとして取得|
|s[ラベル名・番号のスライス]|単独・複数の要素の値をpandas.Seriesとして取得|
|s[boolのSeries]|Trueの要素をpandas.Seriesとして取得|




### boolのSeriesを得る方法の例
|||
|-|-|
|Series >= 数値||
|Series == '文字列'||
|Series != '文字列'||
|Series.isin(['文字列1', '文字列2'])<br>Series.isin([数値1, 数値2])|指定した複数の文字列のいずれかと完全一致。<br>指定した複数の数値のいずれかと完全一致。|
|Series.str.contains(r'pat', na=?)|na(欠損値)をTrueかFalse。デフォルトではNone(行抽出時はエラー)。|
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


<a id="文字列操作"></a>
## 文字列操作

### 基本
|||
|-|-|
|Series.str[2]||
|Series.str[2:9]||
|Series.str.strip()||
|Series.str.join(sep)||
|Series.str.normalize('NFKC')||
|Series.str.zfill(6)|左側をゼロ埋めし、指定した文字数にする|

### 正規表現関連
|||
|-|-|
|Series.str.extract(pat, expand=False)|※ 最初の一つのキャプチャ|
|Series.str.findall(pat)||
|Series.str.replace(pat, repl, regex=True)|※ DataFrame.replace, Series.replaceのほうが便利。|
|Series.str.split(pat, regex=True)|patが初期値のままなら空白で分割される。<br>expand=Trueとすると複数の列に分割。|

|||
|-|-|
|DataFrame.replace({<br>'列名1': {r'^あ': 'aa', r'\d{4}': '8888'},<br>'列名2': {r'(\d+)分': r'\1年'},<br>}, regex=True)<br><br>DataFrame.replace({r'pat1': '', r'pat2': ''}, regex=True)<br>Series.replace({r'pat1': '', r'pat2': ''}, regex=True)|便利な置換手段。<br>DataFrameにもSeriesにも使える。|

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



<a id="基礎集計"></a>
## 基礎集計

### 重要なメソッド
|||
|-|-|
|DataFrame.sum()<br>DataFrame.min()<br>DataFrame.max()<br>DataFrame.mean()<br>DataFrame.median()<br>DataFrame.count()<br>DataFrame.nunique()|sum：列方向の合計を算出。<br>min：列方向の最小値を算出。<br>max：列方向の最大値を算出。<br>mean：列方向の平均値を算出。<br>median：列方向の中央値を算出。<br>count：列方向の非欠損値数を算出。<br>nunique：列方向のユニーク数を算出。<br>※axis=1とすると行方向。<br>Seriesを返す。|
|Series.sum()|合計を算出。<br>スカラー値を返す。|
|DataFrame.sum().sum()|DataFrame.sum()が返したSeriesのsum()を呼ぶことで、総数を得られる。|
|DataFrame.mean()|mean：列方向の平均値を算出。<br>※axis=1とすると行方向。<br>Seriesを返す。|
|DataFrame.median()|median：列方向の中央値を算出。<br>※axis=1とすると行方向。<br>Seriesを返す。|

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
|gb = DataFrame.groupby('列名')|列名で指定した列の値ごとにグルーピングされる。<br>DataFrameGroupByオブジェクトを返す。<br>デフォルトでは指定した列が結果のインデックスになるが、as_index=Falseとするとインデックスにならない。<br>デフォルトでは指定した列に含まれる欠損値NaNは無視されるが、dropna=FalseとするとNaNも一つのキーとして扱われる。|

### DataFrameGroupByオブジェクトとは
グループ名と、そのグループのDataFrameが対になっているイメージ。  
当然、Boolean Indexingでも特定の列の値を指定してDataFrameを取得することはできる。  
特定のカテゴリーだけ選択したいのならBoolean Indexingを使い、全カテゴリーを分割し選択できるようにしたいのならgroupbyメソッドを使うといい。  
ちなみにイテラブルオブジェクトである。

```py
# for文を使うと、グループ名とそのグループのDataFrameを順に取得できる。
for group, df in gb:
    print(group, df.shape)
```

### DataFrameGroupByオブジェクトの主なメソッド
|||
|-|-|
|gb.get_group('グループ名')|グループ名で指定したグループのDataFrameを取得できる。|
|gb.size()|行数|
|gb.sum()|合計(各dfに適用して、結合するイメージ)|
|gb.mean()|平均|
|gb.max()|最大|
|gb.min()|最小|
|gb.median()|中央値|
|gb.std()|標準偏差|
|gb.count()|欠損値ではない要素数|
|gb.describe()|主要な統計量を一括算出|
|gb.agg(func)|任意の関数適用<br>※自作関数(def or lambda)を指定する場合は、引数がarray-likeとなるように作成。 |

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

### 引数ignore_index
ignore_index=Trueとするとインデックスが振り直される(0からの連番)。

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
|Series.apply(pd.Series)|Seriesの各要素のリストにpd.Series()を適用したDataFrameを返す。<br>※Series.map()はDataFrameを返すことができないため、Series.apply()を使う。|
|DataFrame.apply(lambda s: s[1] in s['所在地'], axis=1)|第一引数に適用したい関数を指定。<br>defで定義した関数やラムダ式も指定可能。<br>デフォルトでは各列がSeriesとして関数に渡される(戻り値が新しい列となる)。<br>引数axisを1とすると各行がSeriesとして関数に渡される(戻り値が新しい行となる)。<br>Laxis(0と1の覚え方)。<br>Seriesを引数として受け取れない関数だとエラーになる。<br>指定した関数をDataFrameの各行もしくは各列に適用する。<br>新しいオブジェクト(SeriesかDataFrame)が返される。|

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
|DataFrame.dropna(how='any', subset=None, ignore_index=False)|欠損値を含む行を削除する。<br>※ how='all'とすると、全てが欠損値の行を削除する。<br>※ subset=['列名1', '列名2']とすると、指定列のみが調査対象になる。<br>※ ignore_index=Trueとすると、インデックスが振り直される(0からの連番)。|
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
|index_col|index_col='date'のように、indexとするカラムを列名で指定できる。省略すると新しいindex(0からの連番)が作成される。|
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
df.to_csv(hoge/fuga/piyo.csv',  sep=',', header=True, index=True, encoding=utf-8, mode='w')
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




<a id="クロス集計"></a>
## クロス集計

### 基本
```py
# クロス集計表を作成する関数
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



<a id="メルト"></a>
## メルト

### 基本
横持ちのDataFrameを縦持ちのDataFrameに再構築する(DB的になる)。
```py
# DataFrameを三種類のカラム(id_vars, variable, value)に再構築して返す。
def melt(
    # 対象のDataFrame
    frame,
    # 列名リストを指定。指定した列はid_varsカラムとなり、meltされずにそのまま残る。
    id_vars,
    # 列名リストを指定。指定した列の列名はvariableカラムとなり、値はvalueカラムとなる。
    value_vars,
    # variableカラムの列名を指定(デフォルトはvariable)。
    var_name,
    # valueカラムの列名を指定(デフォルトはvalue)。
    value_name,
) -> DataFrame:
```

### 再構築したDataFrameのイメージ
|id_vars|variable|value|
|-|-|-|
|id_vars指定列(全)|value_vars指定列の列名(1)|value_vars指定列(1)|
|id_vars指定列(全)|value_vars指定列の列名(2)|value_vars指定列(2)|
|id_vars指定列(全)|value_vars指定列の列名(3)|value_vars指定列(3)|
|id_vars指定列(全)|value_vars指定列の列名(4)|value_vars指定列(4)|
|id_vars指定列(全)|value_vars指定列の列名(5)|value_vars指定列(5)|




### 参考サイト
* [pandas.melt — pandas 2.2.2 documentation](https://pandas.pydata.org/docs/reference/api/pandas.melt.html)
* [データフレームを再構築するPandasのMelt()関数のお話し](https://www.salesanalytics.co.jp/datascience/datascience021/)




<a id="その他"></a>
## その他

### 基本的な操作
|||
|-|-|
|df['str1'] + ' in ' + df['str2']||
|df['num1'] * 3 / df['num2']||
|df[['num1', 'num2']] /= 3||
|(df['num'] / 30).astype(str).str[0:3].astype(float)||
|(df['num'] / 30).astype(int)||
|df[2:9]|行名もしくは行番号のスライスで該当した行をDataFrameとして取得。<br>intのスライスは、常に行番号のスライスとして扱われる。つまりintの行名を指定することは不可。|


### ランダムサンプリング
|||
|-|-|
|DataFrame.sample(n=30)|n行だけランダムサンプリング|
|DataFrame.sample(frac=0.1)|行を指定割合だけランダムサンプリング|


### その他
|||
|-|-|
|DataFrame.reset_index()|indexを0からの連番に振り直す。<br>元のindexはデータ列として残る(列名は'index')。<br>drop=Trueとすると、元のindexは削除され残らない。<br>※引数にignore_index=Trueを指定できるメソッドに対しても、こっち(reset_index)を使うほうがいいかも。|
|DataFrame.drop(index=['行名1', '行名2'], columns=['列名1', '列名2'])|DataFrameの行・列を指定して削除する。|
|DataFrame.drop_duplicates()<br>Series.drop_duplicates()|重複行を削除する。<br>※引数(subset、keep)による重複行の扱い方はduplicatedメソッドと同じ。|
|DataFrame.sort_values(by='列名', ascending=False, ignore_index=True)|デフォルトは昇順。降順にするには引数ascendingをFalseにする。<br>na_position='first'とすると、欠損値NaNが先頭に並べられる。デフォルトでは末尾。<br>ignore_index=Trueとするとインデックスが振り直される(0からの連番)。|
|DataFrame.transpose()|転置|
|DataFrame.copy()|dfのコピーを作成(参照の値渡しを防ぐため)。|
|DataFrame.rename(index={'元の行名': '新しい行名'}, columns={'元の列名': '新しい列名'})|行名・列名のいずれかのみを変更したい場合は、引数indexとcolumnsのどちらか一方だけを指定すればよい。|
|Series.shift(2, fill_value='あいうえお')|行方向に値がずれた列を作成する。<br>第一引数にずらし幅を指定。fill_valueにずれた部分に設定する値を指定(省略すると欠損値)。|

### 保留
* ループ処理(iterrows、itertuples)
* 欠損値を前後の値から線形補間(interpolate)
* agg
* pandas.DataFrame.update()※DataFrameの値の更新
