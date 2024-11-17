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


## copilotの回答

hover_data
カーソルを当てたときの表示を変える。

data_frame: グラフに使用するデータフレーム。

x: X軸に使用するカラム名。

y: Y軸に使用するカラム名。

color: データポイントの色を分けるカラム名。

pattern_shape: データポイントのパターンを分けるカラム名。

facet_row: 行ごとにサブプロットを分けるカラム名。

facet_col: 列ごとにサブプロットを分けるカラム名。

facet_col_wrap: 列ごとに分けるサブプロットの最大数。

facet_row_spacing: 行間の間隔。

facet_col_spacing: 列間の間隔。

hover_name: ホバーテキストに表示するカラム名。

hover_data: ホバー時に追加で表示するデータ。

custom_data: カスタムデータをホバーテキストに表示。

text: バーに表示するテキスト。

base: バーの基点。

error_x: X軸方向のエラーバー。

error_x_minus: X軸方向の負のエラーバー。

error_y: Y軸方向のエラーバー。

error_y_minus: Y軸方向の負のエラーバー。

animation_frame: アニメーションを分けるフレーム。

animation_group: アニメーションのグループ化。

category_orders: カテゴリーの順序。

labels: 軸ラベルのカスタマイズ。

color_discrete_sequence: カラーシーケンスのカスタマイズ。

color_discrete_map: カラーのマッピング。

color_continuous_scale: カラーの連続スケール。

pattern_shape_sequence: パターンシーケンスのカスタマイズ。

pattern_shape_map: パターンのマッピング。

range_color: カラーの範囲。

color_continuous_midpoint: カラーの中間点。

opacity: バーの透明度。

orientation: バーの方向（'h'または'v'）。

barmode: バーモード（'relative', 'group', 'overlay'など）。

log_x: X軸を対数スケールにするかどうか。

log_y: Y軸を対数スケールにするかどうか。

range_x: X軸の範囲。

range_y: Y軸の範囲。

text_auto: 自動的にテキストを表示するかどうか。

title: グラフのタイトル。

template: テンプレートのカスタマイズ。

width: グラフの幅。

height: グラフの高さ。


## 引数

```py
def bar(
    # グラフに使用するDataFrame。
    data_frame=None,
    # x軸に使用する列名、または列名リスト。
    x=None,
    # y軸に使用する列名、または列名リスト。
    y=None,
    # 指定した列でデータを色分けする。
    color=None,
    # 指定した列でデータのパターン形状を分ける。
    pattern_shape=None,
    # 列名で指定した値のカテゴリ毎に分割して複数のグラフを作成する(行ごと)。
    facet_row=None,
    # 列名で指定した値のカテゴリ毎に分割して複数のグラフを作成する(列ごと)。
    facet_col=None,
    facet_col_wrap=0,
    facet_row_spacing=None,
    facet_col_spacing=None,
    hover_name=None,
    hover_data=None,
    custom_data=None,
    # 列名を指定。棒グラフにテキスト文字として表示させるデータ。
    text=None,
    base=None,
    error_x=None,
    error_x_minus=None,
    error_y=None,
    error_y_minus=None,

    # アニメーションに使用する列名
    animation_frame=None,
    animation_group=None,
    category_orders=None,
    # x軸、y軸のラベル名を上書きしてカスタマイズできる。
    # 例：labels={'x軸ラベル名': 'x軸カスタマイズ名', 'y軸ラベル名': 'y軸カスタマイズ名'}
    labels=None,
    color_discrete_sequence=None,
    color_discrete_map=None,
    color_continuous_scale=None,
    pattern_shape_sequence=None,
    pattern_shape_map=None,
    range_color=None,
    color_continuous_midpoint=None,
    # バーの透明度(0~1)。
    opacity=None,
    # デフォルト: 縦棒グラフ
    # 'h': 横棒グラフ
    # 'v': 縦棒グラフ?
    orientation=None,
    # 'relative'(デフォルト): 複数の棒が縦に積み上がる
    # 'group': 複数の棒が横に並ぶ
    # 'overlay': 複数の棒が半透明で重なり合う
    barmode="relative",
    log_x=False,
    log_y=False,
    range_x=None,
    range_y=None,
    # 棒グラフにテキスト文字としてデータを表示させる。
    text_auto=False,
    # グラフのタイトル
    title : str | None = None,
    # グラフ書式テンプレートを指定。
    # template='plotly_dark'がかっこいい。
    template=None,
    # グラフの幅(ピクセル単位)。
    width: int | None = None,
    # グラフの高さ(ピクセル単位)。
    height: int | None = None,
) -> go.Figure:
```



