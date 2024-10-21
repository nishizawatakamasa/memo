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
```




### 参考サイト
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