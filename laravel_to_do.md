# laravel

## インストール
カレントディレクトリにlaravelをインストールしてプロジェクトを自動生成
（プロジェクトを作るたびにlaravelをインストールする必要がある）
[コマンド]composer create-project laravel/laravel projectname


## サーバー起動
[コマンド]php artisan serve


## ルーティングの設定
routes/web.php に設定を追加。

app/Http/Controllers ディレクトリに TaskController.php を作成。
[コマンド]php artisan make:controller TaskController 


## データベースの接続設定
.env.example
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=first_db
DB_USERNAME=root
DB_PASSWORD=
DB_CHARSE=utf8mb4          # 追記：文字コード設定
DB_COLLATION=utf8mb4_general_ci   # 追記：照合順序

.env.exampleの設定を.envにコピーする
[コマンド]cp .env.example .env

C:\xampp\htdocs\test-laravel\config\database.php
'charset' => env('DB_CHARSET', 'utf8mb4'),
'collation' => env('DB_COLLATION', 'utf8mb4_general_ci'),

## テーブルを作成する
foldersテーブルのマイグレーションファイルを作成する
[コマンド]php artisan make:migration create_folders_table --create=folders

マイグレーションファイルを実行してテーブルを作成する
[コマンド]php artisan migrate




## モデルを作成する
[コマンド]php artisan make:model Folder

## シーダーを作成する
[コマンド]php artisan make:seeder FoldersTableSeeder


## シーダーを実行してDBにデーターを挿入する
Composerをオートロードしてシーダーを認識させる
[コマンド]composer dump-autoload
シーダーでDBにデータを挿入する
[コマンド]php artisan db:seed --class=FoldersTableSeeder


## コントローラーにロジックを記述
$folders = Folder::all();

## テンプレートの作成
index.blade.php
styles.css

<link rel="stylesheet" href="../../../public/css/styles.css">



※注意点
記載ミス↓
Tasks::where('folder_id', '=', $folder->id)->get();
正解↓
Task::where('folder_id', '=', $folder->id)->get();
最強版↓
$folder->tasks()->get();


lang/が無い場合、以下のコマンドを打って作成
[コマンド]php artisan lang:publish

resources/lang/en
resources/lang/ja



入門07
<form action="{{ route('folders.edit', ['id' => $folder_id]) }}" method="post">

入門11
php artisan make:provider RouteServiceProvider
[コマンド]public const HOME = '/';

入門11
use App\Providers\RouteServiceProvider;
protected $redirectTo = RouteServiceProvider::HOME;

入門11
use Laravel\Sanctum\HasApiTokens;
[コマンド]composer require laravel/sanctum
[コマンド]php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

database/migrations/yyyy_mm_dd_000001_create_personal_access_tokens_table.php ファイルは不要なので削除する。

※HomeController
use App\Providers\RouteServiceProvider;

test1234

TaskController.php
function showEditForm
 function edit
->tasks()->findOrFail($task_id);
---------------------------------------------

管理者として実行したPowerShellで
[コマンド]net stop mysql82
(82)はmysqlのバージョン