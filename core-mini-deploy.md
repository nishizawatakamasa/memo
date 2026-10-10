
* SSH接続用に鍵認証の準備
  * 公開鍵と秘密鍵のペア(単なる割符)を作成
  * 公開鍵をサーバーに登録
  * 秘密鍵を使った目標サーバーへの接続、の設定を名前付きで保存しておき、簡単に呼べるようにしておく。



鍵認証によるSSH接続用のを準備
VS Code GUI
【PC → サーバー】 SSHログインして初期ディレクトリと .env を手作業で作る
【PC（Herd環境）】 プロジェクトにDeployerを導入・設定する
【GitHub】 秘密鍵などの接続情報を登録する
【デプロイ実行】 Git Push して自動デプロイを確認する



SSH接続では接続ごとに使い捨ての共通鍵が自動で作られ、通信データを共通鍵暗号で暗号化する。
パスワード認証は安全性で劣る上に面倒なので、鍵認証で行うのがデファクトスタンダード。
鍵認証用の公開鍵と秘密鍵のペア(単なる割符)を作成するコマンド。  
`ssh-keygen -t ed25519 -C "hotaka-nishizawa" -f ~/.ssh/id_ed25519_coreserver_m45`  
* ssh-keygen：SSH鍵を生成する
* -t ed25519：鍵の方式を指定。Ed25519がデファクトスタンダード。-tの意味はtype
* -C "hotaka-nishizawa"：鍵に付けるコメント。単なる区別用。サーバー内で見れるので、「どこからの接続か」が分かるとよい。
* -f ~/.ssh/id_ed25519_coreserver_m45：鍵ファイル名を指定。命名は自由だが「id_ed25519_xxx」のxxxに用途などを入れることが多い。既存ファイルを上書きしないように注意。-fの意味はfile。自PC内で読むファイル名なので、「どこへの接続か」が分かるとよい。


鍵作成時に、パスフレーズを聞かれる。パスフレーズは「秘密鍵そのものにかける追加のパスワード」であり、二度同じものを入力する※入力中は画面に文字が表示されない。設定した場合、秘密鍵を使う段階でパスフレーズ入力を求められるようになる。パスフレーズを設定しないなら、二度とも空入力でいい。

これで ~/.ssh/ の中に2つのファイルができる
* id_ed25519_deploy.pub（公開鍵：サーバーに渡す）
* id_ed25519_deploy（秘密鍵：絶対に他人に見せてはいけない。）


サーバーに対し、SSH接続元のIP許可をとる。

パスワード認証でサーバーにSSH接続し、ログインする。
`ssh ユーザー名@サーバー名.coreserver.jp -p 22`
パスワードを聞かれるので、コアサーバーのログインパスワードを入力
ログインに成功すると、サーバーの操作画面（[ユーザー名@... ~]$）になる


ログインした状態のまま、以下の処理を実行する。

ホームディレクトリに.ssh フォルダを作成
`mkdir -p ~/.ssh`
* mkdir -p は「そのディレクトリがなければ作る。すでにあれば何もしない」という意味。
* ~ は「現在ログインしているユーザーのホームディレクトリ」

権限を700にする(自分だけがアクセスできる)
`chmod 700 ~/.ssh`
* chmod は、ファイルやディレクトリの権限を変更する

PC側で、作った公開鍵の中身をクリップボードにコピー
`Get-Content ~/.ssh/id_ed25519_deploy.pub | Set-Clipboard`

公開鍵をauthorized_keysに書き込む
`echo 'ここに公開鍵の文字列をそのまま貼り付ける' >> ~/.ssh/authorized_keys`
* リダイレクト記法 ※書き込み先のファイルがなければ自動で作成される
  * `echo "AAA" > file.txt`：file.txt を上書き
  * `echo "BBB" >> file.txt`：file.txt の末尾に1行追加
* authorized_keysは「このアカウントにSSHログインしてよい公開鍵の名簿」用のファイル。
* ファイルとしてはただのテキストだが、「1行＝1つの公開鍵」という特殊なルールで使われる。
* authorized_keysに既存の鍵を残したまま、新しい公開鍵を1行追加する必要があるので`>>`を使う。

権限を600にする(自分だけが読み書きできる) 
`chmod 600 ~/.ssh/authorized_keys`


PC側の~/.ssh/config
SSH接続の設定にニックネームを付けて、その名前で簡単に使えるようにしておく。
```
Host coreserver # 登録名。いわばこの接続設定につけたニックネーム。
    HostName m45.coreserver.jp # 実際に接続するサーバーの住所。
    User kokoronoki # サーバーにログインするときのユーザー名
    IdentityFile ~/.ssh/id_ed25519_deploy # SSH認証に使う秘密鍵の場所
    Port 22 # SSH接続に使うポート番号
    TCPKeepAlive yes # 接続が生きているか確認しながら、接続状態を維持する設定。
    IdentitiesOnly yes # 指定した IdentityFile の鍵を基本的に使って認証してね、という指定
```


ここからのssh接続はコマンドではなく、VS CodeのRemote SSHで GUI作業に寄せる




プロジェクトルートで Deployer をインストール
`composer require --dev deployer/deployer`

プロジェクトのルートディレクトリに deploy.php(Deployerの設定) を作成


サーバー側で公開フォルダの場所が固定されている場合、サブドメインを作るとpublic_html/sub.example.comのようなフォルダが自動生成されることがある。この場合はシンボリックリンクを作って解決する。

まず、サーバーが自動生成したフォルダを削除。

そこからcurrent/publicへのシンボリックリンクを作る。
`ln -s /apps/jimonoyasai/current/public /home/user/public_html/sub.example.com`
* コマンド解説：ln -s 飛ばす先 アクセスされる場所
* ln：link（リンク）
* -s：symbolic（シンボリック）
これで、「サーバーは sub.example.com を見に行っているつもりだが、実際は current/public が表示される」という状態が作れる。




ローカルPCの .env.production を、サーバーの shared/.env にコピーするだけ。

2. コマンドでサーバーに転送する
ローカルPCのターミナルで、プロジェクトのディレクトリにいる状態で以下のコマンドを実行します。
（※ アプリ名 の部分をご自身のアプリ名に書き換えてください）
code
Bash
scp -i ~/.ssh/id_ed25519_deploy .env.production kokoronoki@m45.coreserver.jp:~/apps/jimonoyasai/shared/.env









      # webfactory/ssh-agent は、秘密鍵をランナーの ssh-agent（鍵をメモリに保持する仕組み）へ登録します。
      # このあと ssh / rsync / Deployer が、パスワードなしでコアサーバーへ入れます。
      #


コアサーバーに対し、


SSH接続元のIP許可をとる。

GitHub Actionsで立ち上げた仮想マシンからコアサーバーにアクセスするための下準備。
コアサーバーのサイトからAPI KEYを発行、コピーして控えておく

API通信で、仮想マシンIP（毎回変わる）の許可をとるために必要。

コアサーバーの API KEYと秘密鍵をyml内で使うために、

Step 8. GitHub リポジトリに Secrets を登録する
GitHub Actions がサーバーに安全にアクセスできるよう、鍵や接続情報を登録します。
対象のリポジトリをブラウザで開きます。
Settings（設定）→ Secrets and variables → Actions をクリックします。
「New repository secret」 ボタンを押し、以下の 5つ を1つずつ登録します。
Secret 名	設定する値

SSH_PRIVATE_KEY	秘密鍵 の中身（取得方法は下を参照）
CORESERVER_API_KEY	コアサーバーの API KEY
※秘密鍵の取得方法（ローカルPCのターミナルで実行）

cat ~/.ssh/id_ed25519_deploy
# 表示された `-----BEGIN OPENSSH PRIVATE KEY-----` から `-----END OPENSSH PRIVATE KEY-----` までを漏れなくコピー




Step 9. ワークフローファイル（.github/workflows/ci.yml）を作成する
本番の手順はリポジトリの `.github/workflows/ci.yml` が本体です。下は学習用の短い例です。
プロジェクトのルートディレクトリに、以下のフォルダ構造でファイルを作成します：
.github/workflows/ci.yml




Step 10. コミットして Push（初デプロイ！）
ローカルPCのプロジェクトルートで Git コミットし、GitHub にプッシュします。




デプロイ状況の確認
GitHub リポジトリの 「Actions」 タブを開きます。
「Deploy to CORESERVER」というワークフローが黄色いアイコン（実行中）で動いているはずです。
クリックするとリアルタイムログが見られます。
すべて通過して 緑色のチェックマーク（Success） になればデプロイ成功です！
成功したら、ブラウザでサブドメインのURLを開いて表示を確認してみてください。




初回の「テーブル初期データ（Seeder）」を入れたい場合
artisan:migrate はテーブルを作るだけなので、もし初期データ（管理者ユーザーの作成など、db:seed）を実行したい場合は、初回のデプロイ成功後に1回だけローカルPCから以下のコマンドを実行すればOKです。
code
Bash
# サーバー上で seeder を実行するコマンド
ssh -i ~/.ssh/id_ed25519_deploy kokoronoki@m45.coreserver.jp "/usr/local/bin/php82cli /virtual/kokoronoki/apps/アプリ名/current/artisan db:seed --force"




------------------------------------
手動マイグレーション
ssh -i ~/.ssh/id_ed25519_deploy kokoronoki@m45.coreserver.jp "/usr/local/bin/php84cli /virtual/kokoronoki/apps/jimonoyasai/current/artisan migrate --force"



手動シーダー
ssh -i ~/.ssh/id_ed25519_deploy kokoronoki@m45.coreserver.jp "/usr/local/bin/php84cli /virtual/kokoronoki/apps/jimonoyasai/current/artisan db:seed --force"



------
次回からのデプロイで「自動マイグレーション」させる設定
Deployer v7 では安全のため、マイグレーションは「明示的にフックする」仕様になっています。
deploy.php の末尾に以下の 1行を追加 しておくだけで、次回以降の GitHub push ではマイグレーションも完全自動で走るようになります。
deploy.php の末尾に追加する1行：
code
PHP
// 本番公開（symlink）の直前に自動でマイグレーションを実行する
before('deploy:symlink', 'artisan:migrate');


------------------
解決手順：ローカルの CSV ファイルをサーバーへ転送する
ローカルPCのターミナルで、プロジェクトのルートディレクトリにいる状態で以下の 2つのコマンド を実行してください。
1. サーバー側に csv フォルダを作成する
code
Bash
ssh -i ~/.ssh/id_ed25519_deploy kokoronoki@m45.coreserver.jp "mkdir -p ~/apps/jimonoyasai/shared/storage/app/csv"
2. ローカルにある CSV ファイルをサーバーへコピー（転送）する
code
Bash
scp -i ~/.ssh/id_ed25519_deploy -r storage/app/csv/* kokoronoki@m45.coreserver.jp:~/apps/jimonoyasai/shared/storage/app/csv/


---------------------
ssh -i ~/.ssh/id_ed25519_deploy kokoronoki@m45.coreserver.jp "/usr/local/bin/php84cli /virtual/kokoronoki/apps/jimonoyasai/current/artisan migrate:fresh --seed --force"


-----------------------------
# ローカルの storage/app/public の中身を丸ごとサーバーへ転送
scp -i ~/.ssh/id_ed25519_deploy -r storage/app/public/* kokoronoki@m45.coreserver.jp:~/apps/jimonoyasai/shared/storage/app/public/
