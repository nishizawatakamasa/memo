# pandas覚書

## 基本
|||
|-|-|
|df.at['行名', '列名']|※ 選択した要素の値を取得・変更。<br>※ 変更時にはスカラー値を代入。|
|df.loc[行名のリストかスライス, '列名']<br>df['列名']\(全ての行を選択する場合の簡潔な書き方)|※ 選択した列をSeriesとして取得・変更・追加。<br>※ 変更、追加時にはスカラー値、リスト、NumPy配列、Series(indexを合わせて代入されることに注意)を代入。|
|df.loc[boolのSeriesか行名リストか行名スライス, 列名リストか列名スライス]|※ 選択した範囲をDataFrameとして取得・変更。<br>※ 行や列をリストで指定した場合はその順番で選択される。<br>※ 変更時にはスカラー値、二次元リスト、二次元NumPy配列を代入。|
|df[boolのSeries]\(全ての列を選択する場合の簡潔な書き方)|※ bool値を要素とするSeriesでTrueの行をDataFrameとして取得。<br>※ ブールインデックスと言う。<br>※※ 複数のboolのSeriesに~&\|を適用し、複数条件でDataFrameを取得することも可能。<br>※※ 優先順位が高い順から、~(not)、&(and)、\|(or)<br>※※ 比較演算子を使うときは括弧で括る。<br>※※ 優先したい処理も括弧で括る。<br>|

## boolのSeriesを得る方法の例
|||
|-|-|
|Series >= 数値||
|Series == '文字列'||
|Series != '文字列'||
|Series.isin(['文字列1', '文字列2'])|※ 指定した複数の文字列のいずれかと完全一致。|
|Series.isnull()||
|Series.notnull()||
|Series.str.contains(pat, flags=0, na=?)|※ na(欠損値)をTrueかFalse。デフォルトではNone(行抽出時はエラー)。|
|DataFrame.duplicated(subset=['列名'], keep=False)|※ 重複行がTrue。|

## 文字列操作
|||
|-|-|
|Series.str.strip()||
|Series.str[2]||
|Series.str[2:9]||
|Series.str.normalize('NFKC')||
|Series.str.join(sep)||

## 文字列操作(正規表現関連)
|||
|-|-|
|Series.str.extract(pat, flags=0, expand=False)|※ 最初の一つのキャプチャ|
|Series.str.findall(pat, flags=0)||
|Series.str.replace(pat, repl, flags=0, regex=True)|※ DataFrame.replace, Series.replaceのほうが便利。|
|Series.str.split(pat, regex=True)|※ 初期値空白|

## その他

|||
|-|-|
|df['str1'] + ' in ' + df['str2']||
|df['num1'] * 3 / df['num2']||
|df.drop(columns=['state', 'point'])||
|pd.concat([df1, df2])||
|Series.astype(str)||
|df.transpose()|※ 転置|
|DataFrame.fillna(value)<br>Series.fillna(value)|※ 欠損値をvalueに置換|
|DataFrame.drop_duplicates(subset=['列名'])|※ 指定した列の重複行を、最初の行だけを残して削除|
|df[2:9]|※ 行番号のスライスで該当した行をDataFrameとして取得。|
|DataFrame.sample(n=30)|※ n行だけランダムサンプリング|
|DataFrame.sample(frac=0.1)|※ 行を指定割合だけランダムサンプリング|
|DataFrame.map(lambda x: hex(int(x)))<br>Series.map(lambda x: func(x, 5))|※ na_actionを'ignore'とすると、NaNは関数に渡されずに結果がそのままNaNとなる。|
|df.sort_values(by='列名', ascending=False, ignore_index=True)|※ デフォルトは昇順。降順にするには引数ascendingをFalseにする。<br>※ na_position='first'とすると、欠損値NaNが先頭に並べられる。デフォルトでは末尾。<br>※ ignore_index=Trueとするとインデックスが振り直される。|
|DataFrame.replace({<br>'列名1': {r'^あ': 'aa', r'\d{4}': '8888'},<br>'列名2': {r'(\d+)分': r'\1年'},<br>}, regex=True)<br><br>Series.replace(r'(.+)abc(.*)', r'\2xx\1', regex=True)|※便利な置換手段。<br>※DataFrameにもSeriesにも使える。|


* 追加予定？の項目
    * グループ化
    * マージ
