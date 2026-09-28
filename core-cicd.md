# コアサーバーCI/CD

* 目次
    * [全容](#全容)
    * [.yml](#.yml)
    * [GitHubActions](#GitHubActions)



<a id="全容"></a>
## 全容
"
全体のデプロイフロー（5つのステップ）
code
Text
[開発者] 
   │ git push
   ▼
[GitHub Actions (無料の強力なUbuntu環境)]
   │ ① PHPライブラリ取得 (composer install)
   │ ② Reactのビルド (npm run build ➔ JS/CSSの静的ファイルを生成)
   │ ③ TreeliteのCコードをLinuxバイナリにコンパイル (gcc -O3)
   │ ④ Deployer を起動
   ▼ (SSH接続)
[月300円のレンタルサーバー]
   │ ⑤ ファイル転送 ➔ シンボリックリンクを一瞬で切り替え (完了！)


Deployerがやってくれている「一番便利なこと」
Deployerがなぜ使われるかというと、**「ゼロダウンタイム（無停止）デプロイ」と「一瞬でのロールバック（巻き戻し）」**ができるからです。
レンタルサーバーの中は、Deployerによって以下のようなディレクトリ構造になっています。
code
Text
/home/user/app/
├── current -> /home/user/app/releases/3  (← Web公開用シンボリックリンク)
│
├── releases/
│   ├── 1/ (過去のバージョン)
│   ├── 2/ (前回のバージョン)
│   └── 3/ (★今回の最新バージョン：GitHubから送られてきた完成品)
│
└── shared/ (バージョンをまたいで使い回す共通ファイル)
    ├── .env (DBのパスワードなどの設定ファイル)
    └── storage/ (ユーザーがアップロードした画像やログなど)
Deployerの神ムーブの流れ：
安全な場所で準備: releases/3 という新しいフォルダを作り、そこにGitHub Actionsからビルド済みのファイルを流し込む。
共有ファイルの結合: shared/.env などを releases/3 にリンクさせる。
DBマイグレーション: php artisan migrate --force を実行。
一瞬で切り替え（アトミックデプロイ）:
Web公開用のリンク（current）の向き先を、releases/2 から releases/3 に0.001秒でカチッと切り替える。
古いゴミ掃除: 古い releases/1 を自動で削除（通常は過去3〜5世代だけ残す）。
もし新しいバージョン（3）でバグがあっても、コマンド一発でリンクを 2 に戻す（ロールバックする）だけで一瞬で復旧できます。



git pushや手動デプロイをトリガーにGitHub Actionsを起動
GitHubがUbuntuの仮想マシン(空っぽの作業フォルダ)を一つ起動。

仮想マシン内であれこれと処理
deploy.yml
GitHub は .github/workflows/ の中の YAML を見つけると、自動実行の手順として使います。

仮想マシン内の処理終了

仮想マシンは基本的に破棄









仮想マシン内であれこれと処理

GitHubからリポジトリ取得
uses: actions/checkout@v4
が実質的に
git clone ...
git checkout ...
をやってくれている。

GitHubが用意しているUbuntuの仮想マシンに、開発でよく使うツールがあらかじめ大量にインストールされている
依存関係のインストールやビルドと言った重い処理をGitHubの強力なサーバーでぶん回す。

Deployerが、完成したファイル一式(素のPHPサイト同然)をSSH通信でサーバーへ転送する。
その際、rsyncによる差分転送が行われる。
Deployer: フォルダの世代管理や、シンボリックリンク切り替えを行う司令塔。
SSH: 通信を暗号化するので安全。サーバー上でコマンドを直接実行できるから速い。
rsync: 差分転送ツール。2回目以降の転送では、前回から変更された差分ファイルだけを送る。

備考:Linuxの世界にあるすべてのファイルはハードリンクに過ぎない。ファイルの実体データはストレージのデータブロック領域にある。
各 releases/n/ は単体で完結して動くフルセットだが、あくまでファイルを指すハードリンク(名札)の集まりに過ぎず、サーバーのSSD容量をほぼ消費しない。Linuxは、その実体を指すハードリンク数が0になった時に初めてデータを完全消去する。



<a id=".yml"></a>
## .yml



.yml(ヤムル)は、設定ファイルなどでよく使われる「人間が読み書きしやすい」データ記述フォーマット。




基本文法から、GitHub Actions で仮想マシン（ランナー）を起動して処理を動かす具体的な書き方までを分かりやすく解説します。
1. YAMLの基本文法・ルール
YAMLには絶対に守らなければならない基本ルールがいくつかあります。
① インデントは「半角スペース」のみ
タブはエラー。
通常は半角スペース2つでネストを表す。
② コロン : やハイフン - の後ろには「半角スペース」が必須

2. YAMLの基本構文
(1) 基本は「キー: 値」（辞書 / マッピング）
code
Yaml
name: Tanaka
age: 25
is_student: false
文字列はクォート（" や '）で囲まなくてもOKですが、特殊文字を含む場合や文字列として明示したい場合は囲みます。
(2) リスト（配列）
ハイフン - と半角スペースを使います。
code
Yaml
fruits:
  - Apple
  - Banana
  - Orange
(3) 階層構造（組み合わせ）
辞書の中にリストを入れたり、その逆も可能です。
code
Yaml
user:
  name: Tanaka
  hobbies:
    - Programming
    - Reading
(4) 複数行のテキスト（改行を保持する）
パイプ | を使うと、改行を含めた文字列をそのまま書けます（シェルコマンドを複数書くのによく使われます）。
code
Yaml
script: |
  echo "Hello"
  echo "World"
(5) コメント
# から行末までがコメントになります。
code
Yaml
# これはコメントです
name: Tanaka


GitHub Actions では、リポジトリの .github/workflows/xxxx.yml に処理を記述します。


GitHubにコードがプッシュされたら、Ubuntu仮想マシンを起動してテストを実行する例です。
```yml
      # ステップ4: 複数行のシェルコマンドを実行する
      - name: Run Tests and Build
        run: |
          echo "Starting tests..."
          npm test
          echo "Building project..."
          npm run build
```

- name: Run script with ENV
  env:
    MY_SECRET: ${{ secrets.MY_API_KEY }} # GitHubのSecretsから呼び出し
  run: echo "Env set!"


```yml

# name: 作業名。日本語でも問題ない。
name: CI / Deploy

# いつ実行するかのトリガー
on:
  # 指定ブランチにコミットが push されたとき。
  push:
    # ブランチ指定。※Mergeも、中身はpush。
    branches: [main]
  # 手動で実行したとき。GitHub の Actions 画面から「Run workflow」で実行できる。
  workflow_dispatch:

# このワークフローが GitHub API で何をしてよいかの権限設定。
# 書いておかないと、デフォルト権限が広すぎることがある。基本的には最小にする。
permissions:
  # リポジトリのコードを読むだけ。書き込みや Secrets 以外の操作はしない。
  contents: read

# 同じグループのジョブが同時に何個も走らないようにする。
# グループ名が同じなら「同じ本番向けの一連の作業」とみなす。
concurrency:
  group: deploy-production
# デプロイ実行中に新しいデプロイが来たとき、実行中のデプロイを強制終了するかどうか。
#   true  だと、強制終了し、新しいデプロイを開始
#   false だと、実行中のデプロイを最後までやってから、新しいデプロイを開始。
# デプロイの途中で殺すと、サーバー側のロックや中途半端なリリースが残りやすい
  cancel-in-progress: false

# ジョブ（具体的な処理の単位）
jobs:
  # ci: 検査から本番反映まで、同じランナーで直列に進めます。
  ci:
    name: 検査して本番へ反映する
    # if: このジョブを動かす追加条件です。
    # github.ref は今のブランチです。refs/heads/main が main です。
    # 手動実行はどのブランチでも押せるので、main 以外では何もしません。
    # && は「左も右も真」。|| は「どちらかが真」。
    if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'workflow_dispatch')
        # ★ ここで仮想マシンのOSを指定して起動！
    # jobs の中の runs-on で起動するOS（仮想環境）を指定
    # これを指定すると、GitHubが裏で専用の仮想マシン（ランナー）を立ち上げてくれます。
#     主なOS指定：
# ubuntu-latest （最も一般的で高速、こだわりがなければこれ）
# windows-latest （Windows環境が必要な場合）
# macos-latest （iOSアプリのビルドなどが必要な場合）
    runs-on: ubuntu-latest
    # ジョブ全体の制限時間（分）。超えると GitHub がジョブを失敗にする。
    timeout-minutes: 45

    # services: このジョブの横で動かす別コンテナ。

    # ランナー（Ubuntu）自体には MySQL が入っていません。Herd もありません。
    # phpunit.xml が DB_CONNECTION=mysql と DB_DATABASE=chisan_chisho_test を指定しているので、
    # それに合わせた MySQL 8 を立てます。
    # ports: コンテナの 3306 をランナーの 3306 に繋ぎます。アプリからは 127.0.0.1:3306 です。
    # options の health-cmd は「MySQL がクエリを受け付けるまで、最初のステップに進まない」です。
    # 起動直後は接続拒否になるので、10秒おきに最大10回確認します。
    # ステップより先に待たされるので、IP 許可 API はその直後が最速です。
    services:
      mysql:
        image: mysql:8.0
        # 設定。「自身以下」に適用される。下の階層で同じ名前を定義したら上書きされる。
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: chisan_chisho_test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=10


    env:
      DB_CONNECTION: mysql
      DB_HOST: 127.0.0.1
      DB_PORT: 3306
      DB_DATABASE: chisan_chisho_test
      DB_USERNAME: root
      DB_PASSWORD: root
      SSH_HOST: m45.coreserver.jp
      SSH_USER: kokoronoki

    steps:
      - name: 新しい仮想マシンの公開 IP（毎回変わる）を、コアサーバーのファイアウォールへ一時登録する（反映待ちを後ろの作業と重ねる）
        env:
          CORESERVER_API_KEY: ${{ secrets.CORESERVER_API_KEY }}
    #   （シェルスクリプトを実行する）
    # 起動した仮想マシンのターミナルで直接コマンドを叩くのと同じです（LinuxならBashが動きます）。
        run: |
          RUNNER_IP=$(curl -fsS https://api.ipify.org)
          echo "Current Runner IP: $RUNNER_IP"
          HTTP_CODE=$(curl -sS -o /tmp/coreserver_fw.json -w "%{http_code}" -X POST https://api.coreserver.jp/v1/tool/ssh_ip_allow \
            -d "account=${SSH_USER}" \
            -d "server_name=${SSH_HOST}" \
            -d "api_secret_key=${CORESERVER_API_KEY}" \
            -d "param[addr]=$RUNNER_IP")
          echo "API HTTP ${HTTP_CODE}"
          cat /tmp/coreserver_fw.json
          echo
          if [ "$HTTP_CODE" -lt 200 ] || [ "$HTTP_CODE" -ge 300 ]; then
            echo "Failed to allow IP in CORESERVER firewall."
            exit 1
          fi

      - name: 同じ仮想マシンに、このリポジトリをクローンする
        # uses: 既製のAction(再利用できる作業手順)を呼び出して使う。
        uses: actions/checkout@v4

      - name: Node.js 22 を入れて、npm のキャッシュも使う
        uses: actions/setup-node@v4
        # with: 実行するActionに「設定値・引数」を渡す
        with:
          node-version: 22
          cache: npm

      # npm ci
      #   ci = clean install。package-lock.json どおりに node_modules を作り直します。
      #   npm install より再現性が高く、CI 向きです。lock が無いと失敗します。
      - name: package-lock.json どおりに npm パッケージを入れる
        run: npm ci

      # shivammathur/setup-php は、指定バージョンの PHP をランナーに入れる定番アクションです。
      # php-version: '8.4' は、このプロジェクトの composer.json（php ^8.4.1）に合わせています。
      # ここは「この仮想マシン用の PHP」です。本番サーバーの php84cli とは別物です。
      # Feature と Deployer に加え、型検査と Vite も vendor（Ziggy）を見るので、ここで一度だけ入れます。
      #
      # extensions: PHP の拡張機能です。Laravel / Composer がよく必要とするものです。
      #   mbstring  … 日本語などのマルチバイト文字列
      #   xml       … XML 処理（Composer やライブラリが使う）
      #   ctype     … 文字種判定
      #   iconv     … 文字コード変換
      #   mysql / pdo_mysql … MySQL 接続（Feature と、インストール時のパッケージ解決）
      #   bcmath    … 高精度な数値計算
      # coverage: none は「テストカバレッジ用の拡張（xdebug など）を入れない」。
      # 入れると遅くなるためです。
      #
      # tools: deployer:8.0.5
      #   Deployer 本体を、指定バージョンで PATH に入れます。コマンド名は dep です。
      #   composer.lock の deployer/deployer も v8.0.5 です。CI とローカルで版を揃えます。
      #   Deployer は composer.json の require-dev にあります。
      #   あとの composer install --no-dev では vendor/bin/dep は入りません。
      #   だから phar を curl で落とす代わりに、ここで入れます。
      # fail-fast: true は、拡張やツールのインストール失敗を警告で見逃さない設定です。
      - name: PHP 8.4 と Deployer 8.0.5 を入れる
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          extensions: mbstring, xml, ctype, iconv, mysql, bcmath, pdo_mysql
          tools: deployer:8.0.5
          coverage: none
        env:
          fail-fast: true

      # actions/cache/restore は「前回保存した vendor をコピーしてくる」だけです。
      # 同じジョブで vendor を Pest あり → なし と入れ替えるので、ジョブ終了時の
      # 自動保存は使いません。自動保存だと、最後の（Pest なしの）vendor が
      # with-dev の名前で残ってしまうためです。
      #
      # restore-keys は「完全一致が無いとき、近いキャッシュを仮で使う」です。
      # そのあと composer install が差分を直します。
      - name: 前回の vendor（Pest あり）をキャッシュから復元する
        uses: actions/cache/restore@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-with-dev-${{ hashFiles('**/composer.lock') }}
          restore-keys: |
            ${{ runner.os }}-composer-with-dev-

      # ランナー上だけ .env を作ります。本番の shared/.env には送りません（deploy.php で除外）。
      #
      # 直後の composer install で package:discover が .env を見ます。
      # あとの Vite はビルド時に .env の VITE_ で始まる変数を JS へ埋め込みます。
      # .env が無いと VITE_APP_NAME が空になり、画面のアプリ名が Laravel になります。
      # .env.example の APP_NAME はジモノヤサイで、VITE_APP_NAME はそれを参照しています。
      # APP_DEBUG=true は VITE_ で始まらないので、ビルドされた JS には入りません。
      # コピーは1回で足ります。
      - name: .env.example を .env へコピーする（Composer と VITE_APP_NAME 用。本番の .env には送らない）
        run: cp .env.example .env

      # --no-dev は付けません。Pest は require-dev にあるからです。
      # Pint も一緒に入りますが、ここでは回しません。
      # tightenco/ziggy もここに入ります。tsconfig の ziggy-js と Vite の alias が
      # vendor/tightenco/ziggy を指すので、型検査と Vite より先に入れます。
      # それ以外のフラグの意味は、下の --no-dev の composer install と同じです。
      #
      # APP_KEY は example では空です。テストの暗号化（Cookie など）に必要なので、
      # Feature の直前で key:generate します。install の途中の package:discover は空のままで動きます。
      - name: Composer で開発用パッケージ込みでインストールする（Pest が要る。Ziggy もここに入る）
        run: composer install --prefer-dist --optimize-autoloader --no-progress --no-interaction

      # この直後に保存します。あとの --no-dev で vendor が上書きされる前です。
      # 同じ key が既にあるときは保存しません。
      # 型と Vite は、この Pest あり vendor（Ziggy 入り）のまま走ります。
      # Feature のあとに --no-dev へ入れ替えるので、ここで保存しておかないとキャッシュが壊れます。
      - name: vendor（Pest あり）をキャッシュへ保存する
        uses: actions/cache/save@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-with-dev-${{ hashFiles('**/composer.lock') }}

      # npm run types は tsc --noEmit です。TypeScript の型だけを見て、JS ファイルは出しません。
      # npm run build（Vite）は TS/TSX を変換しますが、型の正しさは見ません。
      # 型が違っていても Vite は成功することがあるので、別コマンドで検査します。
      # Feature は withoutVite() なので本番 JS を実行しません。型を Vite・Feature より先に落とし、
      # 壊れた型のままあとの分数を使わないようにします。
      # Composer（Ziggy）より前には置きません。vendor が無いと ziggy-js の型が解決できません。
      # Prettier と ESLint は体裁なので、ここでは回しません。
      - name: TypeScript の型だけ検査する（Vite では型を見ないため。vendor の Ziggy が要る）
        run: npm run types

      # npm run build は package.json の "build": "vite build" です。
      # React / Tailwind などを public/build に静的ファイルとして出力します。
      # 本番サーバーでは npm を回さず、この成果物をアップロードします。
      # 上でコピーした .env の VITE_APP_NAME が、このときに埋め込まれます。
      # vite.config.ts も ziggy-js を vendor/tightenco/ziggy に向けているので、Composer のあとです。
      # 同じジョブで Deployer まで進むので、成果物の受け渡し（artifact）は使いません。
      - name: Vite で本番用の JS/CSS を public/build に出力する
        run: npm run build

      # .env の APP_KEY= を、ランダムな鍵に書き換えます。
      # 本番の shared/.env とは別ファイルです。ここは GitHub のマシンの中だけです。
      - name: テスト用の APP_KEY を生成する
        run: php artisan key:generate --ansi --force

      # composer test:feature は composer.json のスクリプトです。
      # phpunit.xml の Feature スイートだけを、並列 8 プロセスで実行します。
      # Unit は回しません。tests/Browser は phpunit.xml の suite に無いので、ここにもありません。
      # スクリプト側で Composer の「300秒で打ち切る」を無効にしているので、
      # 長いスイートでも Composer には殺されません。上限は上の timeout-minutes です。
      # tests/Pest.php の Feature 向け beforeEach が withoutVite() を呼ぶので、
      # ビルド済み JS は実行しません。型と Vite は上のステップで済ませています。
      # ブラウザテストは本物の JS が要るので、ここでは回しません。
      - name: Feature テストを並列で実行する（Unit とブラウザテストは含まない）
        run: composer test:feature

      # 本番に上げる vendor です。with-dev とは名前の先頭が違うので混ざりません。
      - name: 前回の vendor（本番用・Pest なし）をキャッシュから復元する
        uses: actions/cache/restore@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-no-dev-${{ hashFiles('**/composer.lock') }}
          restore-keys: |
            ${{ runner.os }}-composer-no-dev-

      # composer install は composer.lock に書かれた正確なバージョンで PHP パッケージを入れます。
      # composer update ではないので、突然新しいメジャー版が入ることはありません。
      #
      # --no-dev
      #   require-dev（Pest, Pint, Deployer の composer パッケージなど）を入れません。
      #   本番にテストツールは不要です。容量も時間も減ります。
      #   Deployer 本体は上の setup-php の tools で別途入っています。
      # --prefer-dist
      #   ソース（git clone）より、配布用 zip を優先。通常は速いです。
      # --optimize-autoloader
      #   クラスの自動読み込み表を本番向けに最適化します。
      # --no-progress
      #   プログレスバーを出さない（CI のログが読みやすくなります）。
      # --no-interaction
      #   「はい/いいえ」の質問をしない。CI には人がいないので必須に近いです。
      - name: Composer で本番用パッケージだけ入れる（Pest は入れない）
        run: composer install --no-dev --prefer-dist --optimize-autoloader --no-progress --no-interaction

      - name: vendor（本番用・Pest なし）をキャッシュへ保存する
        uses: actions/cache/save@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-no-dev-${{ hashFiles('**/composer.lock') }}

      # webfactory/ssh-agent は、秘密鍵をランナーの ssh-agent（鍵をメモリに保持する仕組み）へ登録します。
      # このあと ssh / rsync / Deployer が、パスワードなしでコアサーバーへ入れます。
      #
      # ssh-private-key: GitHub Secret に入れた秘密鍵の全文です。
      # 公開鍵のほうはサーバーの ~/.ssh/authorized_keys に登録済み、という前提です。
      # 秘密鍵を YAML に直書きしてはいけません。Git の履歴から消せません。
      - name: 秘密鍵を ssh-agent に登録し、パスワードなしで SSH できるようにする
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # API で IP 許可を出しても、コアサーバー側に反映されるまで数十秒〜数分かかることがあります。
      # すぐ SSH すると拒否されるので、鍵が取れるまで待ちます。
      # 上の Composer・型・Vite・Feature のあいだにも反映は進んでいるので、
      # ここでの待ちは「残り時間」です。埋まっていればすぐ抜けます。
      #
      # mkdir -p ~/.ssh
      #   SSH 用の設定ディレクトリを作ります。-p は「親が無くても作る」「既にあってもエラーにしない」。
      #   ~ はホームディレクトリ（ランナー上では /home/runner など）です。
      # chmod 700
      #   所有者だけ読み書き実行できる、という意味です。SSH はこのディレクトリが緩いと警告したり拒否したりします。
      #
      # set +e
      #   シェルは通常、コマンドが失敗（終了コード非0）した時点でスクリプト全体を止めます（set -e 相当の環境が多い）。
      #   +e はその自動停止をオフにします。待っている間の ssh-keyscan 失敗でステップ全体を落としたくないためです。
      #
      # for i in {1..60}
      #   i を 1 から 60 まで回します。下で sleep 10 なので、最大 60 × 10秒 = 10分です。
      #   コアサーバーの反映は数分かかることがあり、6分だと際どいことがあるためです。
      #
      # ssh-keyscan
      #   サーバーの「ホスト公開鍵」（このサーバー本人です、という指紋）を取得します。
      #   ログインはしません。ファイアウォールがまだ閉じていると、何も返りません。
      #   ホスト名はジョブ env の SSH_HOST です。ファイアウォール API と同じ値です。
      #   -p 22  SSH ポート
      #   -H     ホスト名をハッシュして known_hosts に書く（少しだけ情報を隠す慣習）
      #   2>/dev/null
      #     標準エラー出力（失敗メッセージ）を捨てます。待っている間は失敗が正常なのでログを汚さないためです。
      #     2 は「エラー出力」の番号、> はリダイレクト、/dev/null は「ゴミ箱」です。
      #
      # if [ -n "$SERVER_KEY" ]
      #   -n は「文字列の長さが 0 より大きい」。鍵が取れた = ファイアウォールが開いた、と判断します。
      #
      # >> ~/.ssh/known_hosts
      #   取得した鍵を「知っているホスト一覧」に追記します。
      #   これがないと、SSH が「このサーバー初めて見ますが接続しますか？」と聞いて止まります。CI には人がいません。
      #
      # set -e
      #   成功したので、以降はまた「失敗したら即終了」に戻します。
      # exit 0
      #   このステップを成功で終了します。for の残りは回りません。
      #
      # sleep 10
      #   10秒待ちます。API 反映待ちです。
      #
      # done の外に来たら、60回全部失敗です。exit 1 でワークフローを失敗させます。
      - name: ファイアウォールが開くまで待ち、サーバーの指紋を known_hosts に書く
        run: |
          mkdir -p ~/.ssh
          chmod 700 ~/.ssh
          echo "Waiting for CORESERVER firewall to open..."

          set +e

          for i in {1..60}; do
            SERVER_KEY=$(ssh-keyscan -p 22 -H "${SSH_HOST}" 2>/dev/null)

            if [ -n "${SERVER_KEY}" ]; then
              echo "${SERVER_KEY}" >> ~/.ssh/known_hosts
              echo "Firewall opened! SSH connection successfully established."
              set -e
              exit 0
            fi

            echo "Waiting for firewall update... ($i/60)"
            sleep 10
          done

          echo "Firewall open timed out after 10 minutes."
          set -e
          exit 1

      # dep は setup-php が入れた Deployer 8.0.5 です。
      # deploy は deploy.php に定義された（レシピ込みの）メインタスク名です。
      # prod は deploy.php の host('prod') です。ホストが1つでも、宛先を明示します。
      # -v は verbose（やや詳しくログを出す）。-vvv はさらに詳細で、CI のログが非常に長くなります。
      #
      # このコマンドが、同じリポジトリの deploy.php を読み、SSH でコアサーバーへ反映します。
      # ランナー上の public/build と vendor（--no-dev）が rsync されます。
      - name: Deployer でコアサーバーへアップロードし、本番を切り替える
        run: dep deploy prod -v


```




