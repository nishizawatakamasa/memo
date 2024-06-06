# pandas覚書

## 1. 選択操作
### 基本
|||
|-|-|
|df.loc[<br>boolのSeriesか行名リストか行名スライス,<br>列名か列名リストか列名スライス<br>]|※ 行名や列名がint型の場合は、指定もint型で。※index値の行名はint型。<br>■第二引数が列名の場合、選択した列をSeriesとして取得・変更・追加できる。<br>・ 変更、追加時にはスカラー値、リスト、NumPy配列、Series(indexを合わせて代入されることに注意)を代入。<br>■第二引数が列名リストか列名スライスの場合、選択した範囲をDataFrameとして取得・変更できる。<br>・ 変更時にはスカラー値、二次元リスト、二次元NumPy配列を代入。<br>■行や列をリストで指定した場合はその順番で選択される。<br>■第一引数がbool値を要素とするSeriesの場合、Trueの行が選択される。<br>・ 複数のboolのSeriesに~&\|を適用し、複数条件で選択することも可能。<br>・ 優先順位が高い順から、~(not)、&(and)、\|(or)<br>・ 比較演算子を使うときは括弧で括る。<br>・ 優先したい処理も括弧で括る。|
|df.at['行名', '列名']|※ 選択した要素の値を取得・変更。<br>※ 変更時にはスカラー値を代入。|
|df['列名']|df.loc[:, '列名']の簡潔な書き方(全行選択)。|
|df[boolのSeries]|df.loc[boolのSeries]の簡潔な書き方(全列選択)。|

### boolのSeriesを得る方法の例
|||
|-|-|
|Series >= 数値||
|Series == '文字列'||
|Series != '文字列'||
|Series.isin(['文字列1', '文字列2'])|※ 指定した複数の文字列のいずれかと完全一致。|
|Series.str.contains(pat, na=?)|※ na(欠損値)をTrueかFalse。デフォルトではNone(行抽出時はエラー)。|
|Series.isnull()||
|Series.notnull()||
|DataFrame.duplicated(subset=['列名'], keep=False)|※ keep=Falseで重複行全てがTrue。|

## 2. 文字列操作
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
|Series.str.split(pat, regex=True)|※ 初期値空白|

|||
|-|-|
|DataFrame.replace({<br>'列名1': {r'^あ': 'aa', r'\d{4}': '8888'},<br>'列名2': {r'(\d+)分': r'\1年'},<br>}, regex=True)<br><br>DataFrame.replace(pat, repl, regex=True)<br>Series.replace(pat, repl, regex=True)|※便利な置換手段。<br>※DataFrameにもSeriesにも使える。|

#### インラインフラグ
|||
|-|-|
|r'(?i)hoge'|re.IGNORECASE<br>re.I|
|r'(?m)hoge'|re.MULTILINE<br>re.M|
|r'(?s)hoge'|re.DOTALL<br>re.S|
|r'hoge(?s:fuga)piyo'|部分適用|
|(?im)|複数フラグ|
|(?ms:)|複数フラグを部分適用|

## 3. その他
### 基本
|||
|-|-|
|pd.concat([df1, df2])|※ ignore_index=Trueとするとインデックスが振り直される(0からの連番)。<br>※ axis=1とすると横方向に連結される。|
|df['str1'] + ' in ' + df['str2']||
|df['num1'] * 3 / df['num2']||
|df[2:9]|※ 行番号のスライスで該当した行をDataFrameとして取得。|
|DataFrame.drop(columns=['列名1', '列名2'])<br>DataFrame.drop(index=['行名1', '行名2'])||
|DataFrame.reset_index(drop=True)|※ indexを振り直す。drop=Trueとすると、元のindexは削除され残らない。|
|DataFrame.drop_duplicates(subset=['列名'])|※ 指定した列の重複行を、最初の行だけを残して削除|
|DataFrame.sort_values(by='列名', ascending=False, ignore_index=True)|※ デフォルトは昇順。降順にするには引数ascendingをFalseにする。<br>※ na_position='first'とすると、欠損値NaNが先頭に並べられる。デフォルトでは末尾。<br>※ ignore_index=Trueとするとインデックスが振り直される(0からの連番)。|
|Series.astype(str)||
|DataFrame.fillna(value)<br>Series.fillna(value)|※ 欠損値をvalueに置換|
|DataFrame.sample(n=30)|※ n行だけランダムサンプリング|
|DataFrame.sample(frac=0.1)|※ 行を指定割合だけランダムサンプリング|
|DataFrame.transpose()|※ 転置|
|DataFrame.map(lambda x: hex(int(x)))<br>Series.map(lambda x: func(x, 5))|※ na_action='ignore'とすると、NaNは関数に渡されずに結果がそのままNaNとなる。|
|DataFrame.values<br>Series.values|データ値属性(NumPy配列)。|
|DataFrame.index.to_list()|行名属性をリスト化したもの。|
|DataFrame.columns.to_list()|列名属性をリスト化したもの。|
|DataFrame.index = ['行名1', '行名2', '行名3']|行名属性へ値を代入。|
|DataFrame.columns = ['列名1', '列名2', '列名3']|列名属性へ値を代入。|
|DataFrame.rename(index={'元の行名': '新しい行名'}, columns={'元の列名': '新しい列名'})|行名・列名のいずれかのみを変更したい場合は、引数indexとcolumnsのどちらか一方だけを指定すればよい。

### pd.read_csv 

※read_clipboard()の内部でもread_csv()が動いており、同じ引数が指定できる。
|主な引数|引数の説明|
|-|-|
|filepath_or_buffer|※重要！ CSVファイルのパスを指定。唯一の必須引数。位置引数でもある。|
|sep='\s+'|※重要！ 区切り文字を指定。デフォルトはカンマ。|
|header=None|※重要！ 列名(columns)の設定。デフォルトでは一行目。Noneにすると0始まりの連番。|
|usecols=[2, 6, 7, 8]|※重要！ 読み込む列を、列名のリストで指定(読み込む列を限定するため、メモリの節約にもなる)。|
|dtype=str|※重要！ カラムのデータ型を指定。|
|nrows=2|読み込む行数（ヘッダー行数を除く）を指定。中身をざっと確認したい時用。|
|encoding|文字コードを指定。|
|index_col||
|on_bad_lines||

### pd.read_excel

|主な引数|引数の説明|
|-|-|
|io|Excelファイルのパスを指定。|
|sheet_name|読み込むシートを指定。0始まりの番号かシート名で指定。デフォルトは0。|
|header=None|pd.read_csvと同様。|
|usecols=[2, 6, 7, 8]|読み込む列をリストで指定する。intのリストの場合は列番号（0始まり）での指定となり、文字列のリストの場合は列名での指定となる。列名がintの場合、その列名を指定することはできない。|
|dtype=str|pd.read_csvと同様。|
|index_col||


#### 参考サイト
* [【保存版】Pandas2.0のread_csv関数の全引数、パフォーマンス、活用テクニックを完全解説する！](https://qiita.com/fujine/items/dbe2f5e4101d6299ff12#encoding)

### 基礎集計関数
#### 参考サイト
* [Pythonのpandas基礎集計関数を初心者向けに図解！](https://pythonbunseki.com/python-basic-function/)

### マージ
|||
|-|-|
|df3 = df1.merge(df2, on='キーとする列名', how='left')|標準的なマージ|
#### 参考サイト
* [pandas.DataFrameを結合するmerge, join（列・インデックス基準）](https://note.nkmk.me/python-pandas-merge-join/)  

### グルーピング
#### 参考サイト
* [pandasのgroupby()でグルーピングし統計量を算出](https://note.nkmk.me/python-pandas-groupby-statistics/)  
* [pandas groupbyでグループ化｜図解でわかりやすく解説](https://www.yutaka-note.com/entry/pandas_groupby)
