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
[GitHub Actions]
   │ ① PHPライブラリ取得 (composer install)
   │ ② Reactのビルド (npm run build ➔ JS/CSSの静的ファイルを生成)
   │ ③ TreeliteのCコードをLinuxバイナリにコンパイル (gcc -O3)
   │ ④ Deployer を起動
   ▼ (SSH接続)
[レンタルサーバー]
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


git push直後のGitHub世界の話
pushを検知してGitHub Actionsが起動。
GitHub Actionsは、リポジトリの .github/workflows/ に置かれたYAMLファイルを読み、その中で指定された処理を実行する。   

大体、GitHub内で仮想マシン(空っぽの作業フォルダ)を起動し、その中であれこれと作業する。
その仮想マシンには、開発でよく使うツールがあらかじめ大量にインストールされている。
作業終了時、仮想マシンは基本的に破棄される。









仮想マシン内でのあれこれ

GitHubからリポジトリ取得
uses: actions/checkout@v4
が実質的に
git clone ...
git checkout ...
をやってくれている。

依存関係のインストールやビルドと言った重い処理をGitHubの強力なサーバーでぶん回す。

Deployerが、完成したファイル一式(素のPHPサイト同然)をSSH通信でサーバーへ転送する。
その際、rsyncによる差分転送が行われる。
Deployer: フォルダの世代管理や、シンボリックリンク切り替えを行う司令塔。
SSH: 通信を暗号化するので安全。サーバー上でコマンドを直接実行できるから速い。
rsync: 差分転送ツール。2回目以降の転送では、前回から変更された差分ファイルだけを送る。

備考:Linuxの世界にあるすべてのファイルはハードリンクに過ぎない。ファイルの実体データはストレージのデータブロック領域にある。
各 releases/n/ は単体で完結して動くフルセットだが、あくまでファイルを指すハードリンク(名札)の集まりに過ぎず、サーバーのSSD容量をほぼ消費しない。Linuxは、その実体を指すハードリンク数が0になった時に初めてデータを完全消去する。

- 実際にやっているsteps
  - 新しい仮想マシンのIP（毎回変わる）を、コアサーバーへ一時登録する（時間かかるので真っ先にやる）
  - 仮想マシンに、リポジトリをクローンする
  - Node.js、npm、PHP、Deployerを入れる
  - .env.example を .env へコピーする（Composer と VITE_APP_NAME 用。）
  - composer install 開発用(テスト用)
  - TypeScript の型検査
  - npm run build
  - テスト用の APP_KEY を生成する
  - Feature テストを実行
  - composer install 本番用
  - 秘密鍵を ssh-agent に登録し、パスワードなしで SSH できるようにする
  - コアサーバーのファイアウォールが開くまで待つ
  - ???????????サーバーの指紋を known_hosts に書く
  - Deployer でコアサーバーへアップロードし、本番を切り替える







<a id=".yml"></a>
## .yml

.yml(ヤムル)は、設定ファイルなどでよく使われる「人間が読み書きしやすい」データ記述フォーマット。  


```yml
# YAMLの基本構文
# 基本は「キー: 値」の辞書
# ハイフン - と半角スペースでリスト
# 階層構造にできる。辞書の中にリストを入れたり、その逆も可能。
# インデントは「半角スペース」のみ。通常は半角スペース2つで表現する。タブはエラー。
# コロン : やハイフン - の後ろには「半角スペース」が必須
user:
  name: Tanaka
  hobbies:
    - Programming
    - Reading
```

```yml

# name: 作業名。日本語でも問題ない。
name: CI / Deploy

# on: いつ実行するかのトリガー
on:
  # 指定ブランチにコミットが push されたとき。
  push:
    # ブランチ指定。※Mergeも、中身はpush。
    branches: [main]
  # 手動で実行したとき。GitHub の Actions 画面から「Run workflow」で実行できる。
  workflow_dispatch:

# permissions:このワークフローが GitHub API 経由で何をしてよいかの権限設定。
# 書いておかないと、デフォルト権限が広すぎることがある。基本的には最小にする。
permissions:
  # リポジトリのコードを読むだけ。書き込みや Secrets 以外の操作はしない。
  contents: read

# concurrency: 同時実行を制御する機能
concurrency:
  # group: ワークフローに付けるグループ名。
  # 同じグループ名のワークフローは同時実行できなくなる。
  group: deploy-production
  # cancel-in-progress: デプロイ実行中に新しいデプロイが来たとき、実行中のデプロイを強制終了するかどうか。
  # true だと、強制終了し、新しいデプロイを開始
  # false だと、実行中のデプロイを最後までやってから、新しいデプロイを開始。
  # デプロイの途中で殺すと、サーバー側のロックや中途半端なリリースが残りやすい
  cancel-in-progress: false

# jobs: 実際に動かす処理のまとまり。1つのワークフローに一つだけ。
jobs:
  # jobsの直下に行う作業を定義する。自由なID名を付ける。複数定義することもできる。
  ci:
    name: 検査して本番へ反映する
    # if: このジョブを動かす追加条件。
    if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'workflow_dispatch')
    # runs-on: 指定すると、GitHubが裏で専用の仮想マシンを立ち上げる。
    # ubuntu-latest （最も一般的で高速、こだわりがなければこれ）
    # windows-latest （Windows環境が必要な場合）
    # macos-latest （iOSアプリのビルドなどが必要な場合）
    runs-on: ubuntu-latest
    # timeout-minutes: ジョブ全体の制限時間（分）。超えると GitHub がジョブを失敗にする。
    timeout-minutes: 45
    # services: ジョブの実行中だけ一緒に立ち上げておく「補助サーバー・外部サービス」を定義する
    services:
      # サービス名
      mysql:
        # 使う Docker イメージ
        image: mysql:8.0
        # コンテナ起動時の環境変数
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: chisan_chisho_test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=10

    # env: 設定。環境変数。自身以下のスコープに適用される。
    env:
      DB_CONNECTION: mysql
      DB_HOST: 127.0.0.1
      DB_PORT: 3306
      DB_DATABASE: chisan_chisho_test
      DB_USERNAME: root
      DB_PASSWORD: root
      SSH_HOST: m45.coreserver.jp
      SSH_USER: kokoronoki

    # steps: 実際にやることを上から順番にリストで並べる
    steps:
      - name: 新しい仮想マシンの公開 IP（毎回変わる）を、コアサーバーのファイアウォールへ一時登録する（反映待ちを後ろの作業と重ねる）
        env:
          CORESERVER_API_KEY: ${{ secrets.CORESERVER_API_KEY }}
        # run: コマンドを実行
        # パイプ | を使うと、改行を含めた文字列をそのまま書ける。
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

      - name: package-lock.json どおりに npm パッケージを入れる
        run: npm ci

      - name: PHP 8.4 と Deployer 8.0.5 を入れる
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          extensions: mbstring, xml, ctype, iconv, mysql, bcmath, pdo_mysql
          tools: deployer:8.0.5
          coverage: none
        env:
          fail-fast: true

      - name: 前回の vendor（Pest あり）をキャッシュから復元する
        uses: actions/cache/restore@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-with-dev-${{ hashFiles('**/composer.lock') }}
          restore-keys: |
            ${{ runner.os }}-composer-with-dev-

      - name: .env.example を .env へコピーする（Composer と VITE_APP_NAME 用。本番の .env には送らない）
        run: cp .env.example .env

      - name: Composer で開発用パッケージ込みでインストールする（Pest が要る。Ziggy もここに入る）
        run: composer install --prefer-dist --optimize-autoloader --no-progress --no-interaction

      - name: vendor（Pest あり）をキャッシュへ保存する
        uses: actions/cache/save@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-with-dev-${{ hashFiles('**/composer.lock') }}

      - name: TypeScript の型だけ検査する（Vite では型を見ないため。vendor の Ziggy が要る）
        run: npm run types

      - name: Vite で本番用の JS/CSS を public/build に出力する
        run: npm run build

      - name: テスト用の APP_KEY を生成する
        run: php artisan key:generate --ansi --force

      - name: Feature テストを並列で実行する（Unit とブラウザテストは含まない）
        run: composer test:feature

      - name: 前回の vendor（本番用・Pest なし）をキャッシュから復元する
        uses: actions/cache/restore@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-no-dev-${{ hashFiles('**/composer.lock') }}
          restore-keys: |
            ${{ runner.os }}-composer-no-dev-

      - name: Composer で本番用パッケージだけ入れる（Pest は入れない）
        run: composer install --no-dev --prefer-dist --optimize-autoloader --no-progress --no-interaction

      - name: vendor（本番用・Pest なし）をキャッシュへ保存する
        uses: actions/cache/save@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-no-dev-${{ hashFiles('**/composer.lock') }}

      - name: 秘密鍵を ssh-agent に登録し、パスワードなしで SSH できるようにする
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

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

      - name: Deployer でコアサーバーへアップロードし、本番を切り替える
        run: dep deploy prod -v
```
