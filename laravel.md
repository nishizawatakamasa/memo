
# Laravel覚書


* 目次
    * [インストール](#インストール)
    * [サーバー起動](#サーバー起動)
    * [データベース等の設定](#データベース等の設定)
    * [XAMPPでMySQLが起動しない時の対処法](#XAMPPでMySQLが起動しない時の対処法)
    * [マイグレーション](#マイグレーション)
    * [モデル](#モデル)
    * [シーダー](#シーダー)



<a id="インストール"></a>
## インストール

### カレントディレクトリにlaravelをインストールしてプロジェクトを自動生成するコマンド 
`composer create-project laravel/laravel [プロジェクト名]`  
(プロジェクトを作るたびにlaravelをインストールする必要がある)  
(基本的にはC:\xampp\htdocs内でコマンドを実行)  

<a id="サーバー起動"></a>
## サーバー起動

### サーバー起動コマンド  
`php artisan serve`

<a id="データベース等の設定"></a>
## データベース等の設定
### .env.example
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=first_db
DB_USERNAME=root
DB_PASSWORD=
DB_CHARSE=utf8mb4          # 追記：文字コード設定
DB_COLLATION=utf8mb4_general_ci   # 追記：照合順序
```

### .env.exampleの設定を.envにコピーする。  
`cp .env.example .env`

### C:\xampp\htdocs\test-laravel\config\database.php
```
'charset' => env('DB_CHARSET', 'utf8mb4'),
'collation' => env('DB_COLLATION', 'utf8mb4_general_ci'),
```

<a id="XAMPPでMySQLが起動しない時の対処法"></a>
## XAMPPでMySQLが起動しない時の対処法

### 他で起動してるMySQLがないか確認

別のMySQLが動いていて、同じポートを使用しているとエラーになる。  
デフォルトのポート「3306」が使われているかどうかの確認は、XAMPP Control Panelの「NetStat」からできる。  

別のMySQLが動いていた場合は終了させる。  
管理者として実行したPowerShellで以下のコマンドを実行。   
`net stop mysql82`  
※82はmysqlのバージョンが8.2の場合。


<a id="マイグレーション"></a>
## マイグレーション

Migrationを使うと、テーブルの作成とテーブル構造の定義ができる。ただしデータを入れることはできない。  
操作がコードとして残るので、データベースのバージョン管理のような役割も果たす。  

### マイグレーションファイルの新規作成コマンド  
`php artisan make:migration create_テーブル名_table --create=テーブル名`  

※テーブル名は「格納したい物の名前の複数形」にするのが一般的な慣習。  
例：  
`php artisan make:migration create_folders_table --create=folders`  


### マイグレーションファイルの解説

```php
<?php
// Illuminateには、デフォルトで使える便利なクラスとメソッドが用意されている
// データベース操作用に、Migrationクラス、Blueprintクラス、Schemaクラスを名前空間でインポートする
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

// Laravel9以降では、Migrationクラスが継承された無名クラスが作成される
return new class extends Migration
{
    // upメソッド
    // データベースに新しいテーブルやカラムなどを作成するメソッド
    public function up(): void
    {
        // Schemaクラスのクラスメソッドcreate()を使用して、テーブルの作成とテーブル構造の定義を行う。
        // Schema::create()の第一引数は作成するテーブルの名前
        // Schema::create()の第二引数はテーブルの構造を定義するための無名関数
        // その無名関数はBlueprintクラスのインスタンスを引数として受け取る(通常は$tableという変数名)
        // 無名関数内でそのインスタンスからインスタンスメソッドにアクセスし、カラムを作成する
        // $table->型名になっているインスタンスメソッド('カラム名');という書き方が基本。
        // 例外として、$table->timestamps();は、1行でcreated_atとupdated_atの2列を作成する。
        Schema::create('folders', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title', 20);
            $table->timestamps();
        });
    }

    // downメソッド
    // テーブルを削除するメソッド
    public function down(): void
    {
        // SchemaクラスのクラスメソッドdropIfExists()を使用してテーブルを削除する。
        // Schema::dropIfExists()の引数は削除するテーブルの名前
        // 指定した名前のテーブルが存在する場合にそのテーブルを削除する。存在しない場合は何もしない。
        Schema::dropIfExists('folders');
    }
};
```

### マイグレーションの実行コマンド  
`php artisan migrate`

### 全てのテーブルを削除し、全てのマイグレーションを再実行するコマンド  
`php artisan migrate:fresh`



<a id="モデル"></a>
## モデル

Modelクラス一つがテーブル一つに対応。  
デフォルトでは、クラスはクラス名の複数形が名前になっているテーブルと紐づく。  
テーブル名を明示的に指定することもできる。  
例：  
```php
protected $table = 'folders';  
```
Modelインスタンスは一行分のデータ、Collectionクラスは複数行分のデータに相当。  
CollectionクラスはModelインスタンスを複数格納する配列のようなクラス。  
データベースの各レコードをModelインスタンスとして操作する。  
Modelインスタンスの各インスタンス変数は、その行のフィールドデータに対応する。  

### モデルの新規作成コマンド  
`php artisan make:model モデル名`  
例：  
`php artisan make:model Folder`  




データ取得時の返り値はModelインスタンスかCollectionクラス。

取得したデータがModelオブジェクトの場合、そのメソッドを使用できます。

インスタンスメソッドと静的メソッドがある





データベース

テーブル
レコード(一行分のデータ)
カラム(一列分のデータ)
フィールド(１つのデータ)

new演算子で新しいインスタンス(レコード)を作成し、編集してsave()でレコードを追加できる。  
### インスタンスメソッド：
これらのメソッドは、Eloquentモデルのインスタンス（レコード）ごとに使用されます。

* ★save(): モデルの変更をデータベースに保存します。既存のデータと異なるかチェックをし、異なる場合のみに更新処理を行う。
* update(array $attributes): モデルの属性を更新し、データベースに保存します。
* ★delete(): モデルをデータベースから削除します。
* refresh(): モデルの属性をデータベースから再取得します。
* toArray(): モデルを配列に変換します。
* getAttribute($key): 指定された属性の値を取得します。
* setAttribute($key, $value): 指定された属性に値を設定します。
* getAttributeValue($key): 指定された属性の値を取得します。
* getAttributeFromArray($key): 指定された属性の値を配列から取得します。
* getOriginal($key = null, $default = null): オリジナルの属性値を取得します（更新前の値）。

### 静的メソッド：
これらのメソッドは、Eloquentモデルクラス自体に対して呼び出されます。
* delete(): モデルをデータベースから削除します。
#### Modelインスタンスを返すメソッド：
* ★find($id): 指定されたIDのモデルインスタンスを取得。
* findOrFail($id): 指定されたIDのモデルインスタンスを取得しますが、見つからない場合は例外をスローします。
* where($column, $value): 指定された条件でモデルインスタンスを検索します。
* findOrNew($id): 指定されたIDのモデルインスタンスを取得しますが、見つからない場合は新しいインスタンスを作成します。
* first(): 最初のモデルインスタンスを取得します。
* firstOrFail(): 最初のモデルインスタンスを取得しますが、見つからない場合は例外をスローします。
* create(array $attributes): 新しいモデルインスタンスを作成し、データベースに保存します。
* updateOrCreate(array $attributes, array $values = []): 指定された条件でモデルを検索し、見つからない場合は新しいモデルを作成します。

#### Collectionクラスを返すメソッド：
* ★all(): 全てのモデルインスタンスを取得
* where($column, $value): 指定された条件でモデルインスタンスを検索し、その結果をCollectionクラスのインスタンスとして返します。
* ★get(): モデルインスタンスのコレクションを取得します。
* pluck($column): 指定されたカラムの値を配列として取得します。
* count(): モデルの数を取得します。
* orderBy($column, $direction): 指定されたカラムでモデルインスタンスをソートし、その結果をCollectionクラスのインスタンスとして返します。

------------------------------------




first(): 最初のレコードを取得します。

https://bonoponz.hatenablog.com/entry/2020/10/06/%E3%80%90Laravel%E3%80%91Eloquent%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%A2%E3%83%87%E3%83%AB%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B


https://tobilog.net/10364/




Modelクラス一つがテーブル一つに対応。  

Modelインスタンス
Collectionクラス


allメソッド（全件取得）
Modelクラス::all() ※戻り値はCollectionクラス

findメソッド（プライマリキーで指定したレコードを取得）
Modelクラス::find(1) ※戻り値はModelインスタンス
Modelクラス::find([1, 2, 3]) ※戻り値はCollectionクラス


Modelクラス::where($column, $value)->get(); ※戻り値はCollectionクラス
Modelクラス::where($column, $value)->first(); ※戻り値はModelインスタンス




①リレーションは関数として定義します。
②１対１の主→従の関係はhasoneメソッドを用いることで定義できます。第一引数に従テーブル、第二引数に外部キー、第三引数に内部キーを指定します。
なお、第二引数、第三引数は省略可能で、省略した場合は外部キーに「モデル名_id」内部キーに「id」が使用されます。


①１対１の従→主の関係はbelongsToメソッドを用いることで定義できます。第一引数に主テーブル、第二引数に内部キー、第三引数に外部キーを指定します。
第二引数、第三引数は省略可能で、省略した場合は外部キーに「モデル名_id」内部キーに「id」が使用されます。

①１対多の主→従の関係では、hasManyメソッドを使用します。第一引数に従テーブル、第二引数に参照先テーブルの外部キー、第三引数に参照元テーブルの内部キーを指定します。
第二・第三引数は、hasOneメソッドと同様のルールで省略が可能です。

①１対多の従→主のリレーションは、１対１で使用したbelongToメソッドを使用します。
使い方も１対１と全く同じなのでそちらをご参照ください。


①多対多を定義するには、belongsToManyメソッドを使用します。
第一引数に関連づけたいeloquentモデル、第二引数に中間テーブル名、第三引数に自分に向けられた外部テーブル、第四引数に相手に向けられた外部テーブルを定義します。
第二引数以降は、省略可能です。

hasoneメソッド、belongsToメソッド、hasManyメソッド、belongsToManyメソッドの戻り値からレコードを取得するには、getメソッドやfirstメソッドが必要。









Laravelでは、関係メソッドの命名に一貫性がありません。

<a id="シーダー"></a>
## シーダー

Seederを使うと、データベースに初期データやテストデータを一斉に挿入できる。

### シーダーファイルの新規作成コマンド  
`php artisan make:seeder シーダー名`  
例：  
`php artisan make:seeder FoldersTableSeeder`  

### シーダーファイルの解説
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
// 使用するモデルクラスをインポート
use App\Models\Folder;
// 日時を取得したいならCarbonクラスをインポートする
use Carbon\Carbon;

class FoldersTableSeeder extends Seeder
{
    // runメソッド
    // Seederを実行したときに処理されるメソッド
    public function run(): void
    {
        // たとえば
        $titles = ['プライベート', '仕事', '旅行'];
        // 上記のタイトルでテーブル行を3つ挿入するループを実行する
        foreach ($titles as $title) {
            // モデルクラスのクラスメソッドcreate()を使用。
            // 新しいモデルインスタンスを作成し、データベースに保存する。
            Folder::create([
                // 配列$titlesの値をtitleに代入する
                'title' => $title,
                // Carbonライブラリで現在の日時を取得してcreated_atに作成日として代入する
                'created_at' => Carbon::now(),
                // Carbonライブラリで現在の日時を取得してupdated_atに更新日として代入する
                'updated_at' => Carbon::now(),
            ]);
        }
    }
}
```

DatabaseSeeder.phpから呼び出して使用できるよう、DatabaseSeederクラスに下記を追加。
```php
// runメソッド内に追加する

$this->call(TasksTableSeeder::class);
```

### シーダーの実行コマンド  
`php artisan db:seed`
Seederを実行すると、対象のテーブルにデータがインサートされる。  
実行コマンドがうまくいかない場合はComposerをオートロードしてから実行し直す。 
### Composerをオートロードしてシーダーを認識させるコマンド
`composer dump-autoload`















## コントローラーにロジックを記述
$folders = Folder::all();


最強版↓
$folder->tasks()->get();


lang/が無い場合、以下のコマンドを打って作成
[コマンド]php artisan lang:publish

resources/lang/en
resources/lang/ja



---------------------------------------------

































