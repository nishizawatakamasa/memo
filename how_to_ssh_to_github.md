# SSH接続で複数のGitHubアカウントを使い分ける方法


## 秘密鍵と公開鍵の生成。
1. ホームディレクトリの直下に.sshディレクトリを作成。
1. ~/.sshができる（C:/Users/ユーザ名/.sshという意味）。
1. .sshディレクトリに移動し、コマンド```ssh-keygen -t ed25519```を実行。

<コマンドの意味>
* ssh-keygen：SSH鍵のペアを作成するコマンドラインツール。。
* -t ed25519：-tフラグは、鍵のペアを生成する際に使用するアルゴリズムを指定。「ed25519」が最善。
### 対話形式で3つの質問に答える
1. ファイル名を指定
    * 任意のファイル名を入力してEnter
    * ※例 id_hoge_ed25519
    * ※既存ファイルと同名だと上書きされてしまうので注意
1. パスフレーズ入力
    * 安全性を高めるためのパスフレーズを入力する場面だが、特別なこだわりがない限り不要。
    * 不要ならそのままEnter。
    * 必要なら任意のパスフレーズを入力してEnter。
    * ※入力した場合、git pushをする度に入力を求められるので面倒！
1. パスフレーズ再入力
    * パスフレーズを入力した場合は再入力してEnter。
    * 入力していない場合はそのままEnter。
    * カレントディレクトリの.sshにSSH鍵のペア（秘密鍵と公開鍵(.pub)）が生成される。
    * 個人的イメージ→秘密鍵：カギ。公開鍵：錠前。


## ホスト名と秘密鍵を紐づける設定。
### ~/.ssh/configファイルを作成し、設定を書き込む。  
※例
```
#hogeアカウント
Host github-hoge #任意のホスト名
  HostName github.com
  IdentityFile ~/.ssh/id_hoge_ed25519 #hogeアカウントの鍵のファイル
  User git
  Port 22
  TCPKeepAlive yes
  IdentitiesOnly yes

#fugaアカウント
Host github-fuga #任意のホスト名
  HostName github.com
  IdentityFile ~/.ssh/id_fuga_ed25519 #fugaアカウントの鍵のファイル
  User git
  Port 22
  TCPKeepAlive yes
  IdentitiesOnly yes
```

## 公開鍵をGithubに登録。
### 手順
1. Settings
1. SSH and GPG keys
1. New SSH key
1. Title欄に分かりやすい名前を入力
1. Key typeは「Authentication Key」にする
1. コマンド cat ~/.ssh/公開鍵ファイル名(.pub) を実行すると公開鍵が発行されるので、これをコピーしてKey欄に貼り付け。
1. Add SSH key

## SSHでの接続
* 例： ```git clone git@<ホスト名>:<アカウント名>/<リポジトリ名>.git```
* 使用する秘密鍵と紐づいているホスト名を使う。
* ~/.ssh/configファイルを参照。


## ディレクトリごとにアカウントを自動的に切り替える設定
### ~/.gitconfigファイルに追記
```
[includeIf "gitdir:C:/hoge/"]
   path = ~/.gitconfig_hoge
```
### ~/.gitconfig_hogeファイルを作成し、設定を書き込む。
```
[user]
	name = サブアカウント名
	email = サブアカウントのメールアドレス
```
この設定をすることで、gitdir:の後ろで設定したディレクトリの中にあるプロジェクトディレクトリでは、~/.gitconfig_hogeファイルの中に記載したアカウントに切り替わる。  
指定したディレクトリ以外では、~/.gitconfigファイルに記載されているアカウントが使用される。

