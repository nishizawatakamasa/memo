# plotly.express覚書

* 目次
    * [真骨頂](#真骨頂)
    * [基本](#基本)


```py
import plotly.express as px
import plotly.graph_objects as go
```

グラフの作成には、ファサードであるplotly.expressの使用が推奨されている(内部的にはplotly.graph_objectsが使用される)。  
plotly.expressでは対応できない場合、plotly.graph_objectsを使用する。

[引数についてのドキュメント](https://plotly.com/python-api-reference/generated/plotly.express.bar)  
[引数についてのブログ](https://money-or-ikigai.com/menu/python/Article/Article146.aspx)  
[oyoyo](https://oyoyonochichi.coolblog.jp/Python/plotly/express_bar.html)


## 引数

### 基本
go.Figureオブジェクトを生成し、showメソッドで描画。  
例：
```py
fig = px.line(x=x, y=f(x), template='plotly_dark')
fig.show()
```

### px.line()
```py
def line(
    data_frame=None,
    # array-likeな数値
    x=None,
    # 関数f(x)
    y=None,
    # Trueにすると線上にマーカーが表示される。
    markers=False,
    # x軸を対数スケールにするかどうか。
    log_x=False,
    # y軸を対数スケールにするかどうか。
    log_y=False,
    # x軸の範囲。2つのintのリスト。デフォルトでは自動スケーリング。
    range_x=None,
    # y軸の範囲。2つのintのリスト。デフォルトでは自動スケーリング。
    range_y=None,
    # グラフのタイトル
    title=None,
    # グラフ書式テンプレートを指定。
    # template='plotly_dark'がかっこいい。
    template=None,
    # グラフの幅(ピクセル単位)。
    width: int | None = None,
    # グラフの高さ(ピクセル単位)。
    height: int | None = None,
) -> go.Figure:
```

### px.bar()
```py
def bar(
    # グラフに使用するDataFrame。
    data_frame=None,
    # 列名、または列名リストを指定。x軸に使用する値(カテゴリ)。
    x=None,
    # 列名、または列名リストを指定。y軸に使用する値(カテゴリ)。
    y=None,

    # グラフのタイトル
    title: str | None = None,
    # 列名を指定。バーに表示するテキスト。
    text=None,
    # Trueにすると、方向に応じてxかyかzの値がテキストとしてバーに表示される。 
    text_auto=False,
    # x軸、y軸のラベル名を上書きしてカスタマイズできる。
    # 例：labels={'x軸ラベル名': 'x軸カスタマイズ名', 'y軸ラベル名': 'y軸カスタマイズ名'}
    labels=None,
    # 列名を指定。この列の値は、ホバーツールチップに太字で表示される。
    hover_name=None,
    # 列名を指定。この列の値は、ホバーツールチップの追加データとして表示される。
    hover_data=None,

    # グラフ書式テンプレートを指定。
    # template='plotly_dark'がかっこいい。
    template=None,
    # 指定した列でデータを色分けする。
    color=None,
    # colorで指定した列の値ごとに色を指定できる。例：{'default': '#ffff00', 'highlight': 'red'}
    color_discrete_map=None,
    # 指定した列でデータのパターン形状を分ける。
    pattern_shape=None,
    # バーの透明度(0~1)。
    opacity=None,

    # バーの方向（'h'または'v'）。
    # デフォルト: 縦棒グラフ
    # 'h': 横棒グラフ
    # 'v': 縦棒グラフ?
    orientation=None,
    # 'relative'(デフォルト): 複数の棒が縦に積み上がる
    # 'group': 複数の棒が横に並ぶ
    # 'overlay': 複数の棒が半透明で重なり合う
    barmode="relative",
    # 列名を指定。バーのベース(基点)となる値。
    base=None,
    # x軸の範囲。2つのintのリスト。デフォルトでは自動スケーリング。
    range_x=None,
    # y軸の範囲。2つのintのリスト。デフォルトでは自動スケーリング。
    range_y=None,
    # グラフの幅(ピクセル単位)。
    width: int | None = None,
    # グラフの高さ(ピクセル単位)。
    height: int | None = None,

    # 列名で指定した値のカテゴリ毎に複数のグラフを作成する(各グラフが図の行として縦に並ぶ)。
    facet_row=None,
    # 列名で指定した値のカテゴリ毎に複数のグラフを作成する(各グラフが図の列として横に並ぶ)。
    facet_col=None,
    # facet_rowで作成した各グラフの間隔を指定。
    # 0から1までの浮動小数点(図全体の高さを1としたときの割合)で指定。
    facet_row_spacing=None,
    # facet_colで作成した各グラフの間隔を指定。
    # 0から1までの浮動小数点(図全体の幅を1としたときの割合)で指定。
    facet_col_spacing=None,
    # facet_colで作成した複数のグラフが、いくつ横に並んだら折り返すかを指定。
    facet_col_wrap: int = 0,

    # 列のカテゴリーの順序を辞書で指定(デフォルトでは出現順)。
    # 辞書のキーは列名、値はカテゴリーの順序を文字列のリストで指定する。
    # 例：category_orders={'野菜': ['ナス', 'ピーマン']}
    category_orders=None,

    # 列名を指定。この列の値がアニメーションフレーム(時間軸)に使用される。
    animation_frame=None,
    # 列名を指定。アニメーションの実行時、この列の値ごとにグループ化される。
    animation_group=None,


    custom_data=None,
    error_x=None,
    error_x_minus=None,
    error_y=None,
    error_y_minus=None,
    color_discrete_sequence=None,
    color_continuous_scale=None,
    pattern_shape_sequence=None,
    pattern_shape_map=None,
    range_color=None,
    color_continuous_midpoint=None,
    log_x=False,
    log_y=False,
) -> go.Figure:


# まだよくわかってない引数(copilotの簡単な解説付き)。
 
# custom_data: カスタムデータをホバーテキストに表示。  
# error_x: X軸方向のエラーバー。  
# error_x_minus: X軸方向の負のエラーバー。  
# error_y: Y軸方向のエラーバー。  
# error_y_minus: Y軸方向の負のエラーバー。   
# color_discrete_sequence: カラーシーケンスのカスタマイズ。  
# color_continuous_scale: カラーの連続スケール。  
# pattern_shape_sequence: パターンシーケンスのカスタマイズ。  
# pattern_shape_map: パターンのマッピング。  
# range_color: カラーの範囲。  
# color_continuous_midpoint: カラーの中間点。  
# log_x: X軸を対数スケールにするかどうか。  
# log_y: Y軸を対数スケールにするかどうか。  
```
