# pandas覚書

## 1. 選択操作
### 基本
|||
|-|-|
|df.loc[<br>boolのSeriesか行名リストか行名スライス,<br>列名か列名リストか列名スライス<br>]|■第二引数が列名の場合、選択した列をSeriesとして取得・変更・追加できる。<br>・ 変更、追加時にはスカラー値、リスト、NumPy配列、Series(indexを合わせて代入されることに注意)を代入。<br>■第二引数が列名リストか列名スライスの場合、選択した範囲をDataFrameとして取得・変更できる。<br>・ 変更時にはスカラー値、二次元リスト、二次元NumPy配列を代入。<br>■行や列をリストで指定した場合はその順番で選択される。<br>■第一引数がbool値を要素とするSeriesの場合、Trueの行が選択される。<br>・ 複数のboolのSeriesに~&\|を適用し、複数条件で選択することも可能。<br>・ 優先順位が高い順から、~(not)、&(and)、\|(or)<br>・ 比較演算子を使うときは括弧で括る。<br>・ 優先したい処理も括弧で括る。|
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
|pd.concat([df1, df2])||
|df['str1'] + ' in ' + df['str2']||
|df['num1'] * 3 / df['num2']||
|df[2:9]|※ 行番号のスライスで該当した行をDataFrameとして取得。|
|DataFrame.drop(columns=['列名1', '列名2'])<br>DataFrame.drop(index=['行名1', '行名2'])||
|DataFrame.reset_index(drop=True)|※ indexを振り直す。drop=Trueとすると、元のindexは削除され残らない。|
|DataFrame.drop_duplicates(subset=['列名'])|※ 指定した列の重複行を、最初の行だけを残して削除|
|DataFrame.sort_values(by='列名', ascending=False, ignore_index=True)|※ デフォルトは昇順。降順にするには引数ascendingをFalseにする。<br>※ na_position='first'とすると、欠損値NaNが先頭に並べられる。デフォルトでは末尾。<br>※ ignore_index=Trueとするとインデックスが振り直される。|
|Series.astype(str)||
|DataFrame.fillna(value)<br>Series.fillna(value)|※ 欠損値をvalueに置換|
|DataFrame.sample(n=30)|※ n行だけランダムサンプリング|
|DataFrame.sample(frac=0.1)|※ 行を指定割合だけランダムサンプリング|
|DataFrame.transpose()|※ 転置|
|DataFrame.map(lambda x: hex(int(x)))<br>Series.map(lambda x: func(x, 5))|※ na_action='ignore'とすると、NaNは関数に渡されずに結果がそのままNaNとなる。|
|DataFrame.values<br>Series.values|データ値属性(NumPy配列)。|


### マージ、グルーピング
#### マージの参考サイト
* [pandas.DataFrameを結合するmerge, join（列・インデックス基準）](https://note.nkmk.me/python-pandas-merge-join/)  
#### グルーピングの参考サイト
* [pandasのgroupby()でグルーピングし統計量を算出](https://note.nkmk.me/python-pandas-groupby-statistics/)  
* [pandas groupbyでグループ化｜図解でわかりやすく解説](https://www.yutaka-note.com/entry/pandas_groupby)
