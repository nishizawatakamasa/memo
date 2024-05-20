
# Laravel覚書


* 目次
    * [インストール](#インストール)
    * [サーバー起動](#サーバー起動)
    * [データベース等の設定](#データベース等の設定)
    * [XAMPPでMySQLが起動しない時の対処法](#XAMPPでMySQLが起動しない時の対処法)
    * [lang](#lang)
    * [マイグレーション](#マイグレーション)
    * [モデル](#モデル)
    * [シーダー](#シーダー)
    * [ルーティング](#ルーティング)
    * [コントローラー](#コントローラー)



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

<a id="lang"></a>
## lang

lang/が無い場合、以下のコマンドを打って作成  
`php artisan lang:publish`  

resources/lang/en  
resources/lang/ja  


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

### データベースにおける用語
* テーブル
* レコード(一行分のデータ)
* カラム(一列分のデータ)
* フィールド(１つのデータ)

### モデルの概要
* Modelクラス1つがテーブル1つに相当。
* Modelインスタンスはレコードに相当。
* Modelインスタンスの各インスタンス変数は、その行のフィールドデータに相当。
* Collectionクラスは複数行分のデータに相当(Modelインスタンスを複数格納する配列のようなクラス)。  
* テーブルの各レコードをModelインスタンスとして操作する。  


※デフォルトでは、クラスはクラス名の複数形が名前になっているテーブルと紐づく。  
※テーブル名を明示的に指定することもできる。  
例：  
```php
<?php
protected $table = 'folders';  
```

### モデルの新規作成コマンド  
`php artisan make:model モデル名`  
例：  
`php artisan make:model Folder`  


### インスタンスメソッドとクラスメソッドの違い
```php
<?php
// インスタンスメソッドは、Modelインスタンス(レコード)から呼び出す。つまり、レコードに対しての処理。
$modelInstance->method();
// クラスメソッドはModelクラス(テーブル)から呼び出す。つまり、テーブルに対しての処理。
ModelClass::method();
```

### インスタンスメソッド
```php
<?php
// new演算子で新しいインスタンス(レコード)を作成。
// 編集(例としてタイトルに入力値を代入)
// レコードの追加や変更を保存(既存のデータと異なる場合のみ更新処理を行う)。
// 'created_at'(作成日)と'updated_at'(更新日)は自動的に追加される。
$modelInstance = new ModelClass();
$modelInstance->title = $request->title;
$modelInstance->save();

// レコードを削除
$modelInstance->delete();
```

### クラスメソッド
```php
<?php
// 全レコードを取得
ModelClass::all(); // 戻り値はCollectionクラス

// 主キーで指定したレコードを取得
ModelClass::find(1); // 戻り値はModelインスタンス。無いときはnullなのでissetで判定。
ModelClass::find([1, 2, 3]); // 戻り値はCollectionクラス。

// 全レコードを取得
ModelClass::get(); // 戻り値はCollectionクラス。
// 最初のレコードを取得
ModelClass::first(); // 戻り値はModelインスタンス。無いときはnullなのでissetで判定。

// 条件を指定してフィルタをかける
ModelClass::where($column, $value)->get(); // 戻り値はCollectionクラス。
ModelClass::where($column, $value)->first(); // 戻り値はModelインスタンス。無いときはnullなのでissetで判定。

// テーブルのレコード数を取得する。戻り値は整数値(int)。
ModelClass::count();

// 新しいインスタンス(レコード)の作成、追加、保存を簡潔に書く方法。
// 'created_at'(作成日)と'updated_at'(更新日)は自動的に追加される。
ModelClass::create([
    'folder_id' => 1,
    'title' => "サンプルタスク {$num}",
]);
```

### リレーションは関数として定義する。
```php
<?php
// 主→従の1対1。
// 引数は従テーブル(Modelクラス)、従キー、主キー
return $this->hasOne(Task::class, 'folder_id', 'id');

// メソッド化の参考
public function tasks()
{
    // 主→従の1対多。
    // 引数は従テーブル(Modelクラス)、従キー、主キー
    return $this->hasMany(Task::class, 'folder_id', 'id');
}

// 従→主の1対1、1対多。
// 引数は主テーブル(Modelクラス)、従キー、主キー
return $this->belongsTo(Folder::class, 'folder_id', 'id');

// hasOne()、hasMany()、belongsTo()の第二、第三引数は省略可能。省略した場合は従キーが「主Modelクラス名_id」、主キーが「id」となる。

// hasMany()の戻り値はIlluminate\Database\Eloquent\Relations\HasManyのインスタンス。
// hasOne()の戻り値はIlluminate\Database\Eloquent\Relations\HasOneのインスタンス。

// hasOne()、hasMany()、belongsTo()、belongsToMany()の戻り値からレコードを取得するには、->get()メソッドや->first()メソッドが必要(HasManyインスタンス等のメソッド)。

// 新しいModelインスタンスを、主Modelインスタンスに関連付けて従テーブルに保存する(保存時に従キーが適切に設定される)。
$hasManyInstance->save($modelInstance);
// 主Modelインスタンスに関連するすべてのModelインスタンスを削除する。
$hasManyInstance->delete();

// 新しいModelインスタンスを、主Modelインスタンスに関連付けて従テーブルに保存する(保存時に従キーが適切に設定される)。
$hasOneInstance->save($modelInstance);
// 主Modelインスタンスに関連するModelインスタンスを削除する。
$hasOneInstance->delete();


// 多対多を定義するには、belongsToManyメソッドを使用します。
// 第一引数に関連づけたいeloquentモデル、第二引数に中間テーブル名、第三引数に自分に向けられた外部テーブル、第四引数に相手に向けられた外部テーブルを定義します。
// 第二引数以降は、省略可能です。
```



### アクセサとミューテタ
|||
|:-|:-|
|Accessor(アクセサ)|データベースからデータを取得する際に自動的に呼び出される処理(メソッド)。|
|Mutator(ミューテタ)|データをデータベースに保存する際に自動的に呼び出される処理(メソッド)。|
```php
<?php

namespace App\Models;
// Attributeクラスをインポート
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Carbon\Carbon;

class Task extends Model
{
    use HasFactory;

    protected $table = 'tasks';

    const STATUS = [
        1 => [ 'label' => '未着手', 'class' => 'label-danger' ],
        2 => [ 'label' => '着手中', 'class' => 'label-info' ],
        3 => [ 'label' => '完了', 'class' => '' ],
    ];

    // アクセサやミューテータのメソッド名をスネークケースにしたものが、Modelインスタンスのインスタンス変数としてアクセス可能になる。
    // インスタンス変数status_labelとしてアクセス可能。
    protected function statusLabel(): Attribute
    {
        // make()メソッドで新しいAttributeインスタンスを生成。その時にget引数を与えるとアクセサを定義でき、set引数を与えるとミューテタを定義できる。
        return Attribute::make(
            // get引数に無名関数を渡す。
            // アロー関数でも可。
            // setでも同様。
            get: function ()
            {
                $status = $this->status;
                // PHPのnull合体演算子
                return self::STATUS[$status]['label'] ?? '';
            }
        );
    }

    // インスタンス変数status_classとしてアクセス可能。
    protected function statusClass(): Attribute
    {
        return Attribute::make(
            get: function ()
            {
                $status = $this->status;
                return self::STATUS[$status]['class'] ?? '';
            }
        );
    }

    // インスタンス変数formatted_due_dateとしてアクセス可能。
    protected function formattedDueDate(): Attribute
    {
        return Attribute::make(
            // get引数にアロー関数を渡す。
            get: fn () => Carbon::createFromFormat('Y-m-d', $this->due_date)->format('Y/m/d')
        );
    }
}

```


















https://bonoponz.hatenablog.com/entry/2020/10/06/%E3%80%90Laravel%E3%80%91Eloquent%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%A2%E3%83%87%E3%83%AB%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B

https://tobilog.net/10364/

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
            // 'created_at'(作成日)と'updated_at'(更新日)は自動的に追加される。
            Folder::create([
                // 配列$titlesの値をtitleに代入する
                'title' => $title,
            ]);
        }
    }
}
```

DatabaseSeeder.phpから呼び出して使用できるよう、DatabaseSeederクラスに下記を追加。
```php
<?php
// runメソッド内に追加する
$this->call([
    FoldersTableSeeder::class,
    TasksTableSeeder::class,
]);
```

### シーダーの実行コマンド  
`php artisan db:seed`
Seederを実行すると、対象のテーブルにデータがインサートされる。  
実行コマンドがうまくいかない場合はComposerをオートロードしてから実行し直す。 
### Composerをオートロードしてシーダーを認識させるコマンド
`composer dump-autoload`


<a id="ルーティング"></a>
## ルーティング

```php
<?php

// ルーティングの基本的な書き方
// 第一引数：URL
// 第二引数：第一引数で指定したURLにアクセスした際に実行したい処理。
Route::get('/', function () {
    return view('welcome');
});


// Laravelでは以下６つのHTTPメソッドをルーティングに指定できる。
// GET・・・データを取得。
// POST・・・データを新規追加。
// PUT・・・データの全体を更新。
// PATCH・・・データの一部を更新(ほぼPUTと同じ)。
// DELETE・・・データを削除。
// OPTIONS・・・使えるメソッド一覧を表示。

// ただし、HTMLフォームで許可されているのはGET(データ取得)とPOST(データ送信)のみ。
// そのため、フォームのPUT,PATCH,DELETE,OPTIONSリクエストを使う場合はPOST送信のフォームの中で@method関数を使用し、擬似的にリクエストを実現させる。

// @method関数の使用例
<form action="{{ route('hoges.destroy', ['id' => $id]) }}" method="POST">
    @csrf
    // これを認識したlaravelは、「<input type="hidden" name="_method" value="DELETE">」を生成し、実際のHTTPリクエストメソッドをオーバーライドする。
    @method('DELETE')
    <button type="submit">削除</button>
</form>


// Routeクラスのクラスメソッド
Route::get() // GETに対応。
Route::post() // POSTに対応。
Route::put() // PUTに対応。
Route::patch() // PATCHに対応。
Route::delete() // DELETEに対応。
Route::options() // OPTIONSに対応。
Route::middleware()


// ルーティング定義の例
// URLの{id}や{task_id}のような部分はルートパラメータという。
// ルートパラメータには任意の値が入り、第二引数で指定されたメソッドが実行される際に引数として渡される(例えば{id}の値は$idとして、{task_id}の値は$task_idとして)。
// 第二引数に、実行されるコントローラー(クラス)とそのメソッドを配列で指定している。
// コントローラーは完全修飾クラス名で指定する必要がある(::classというclassキーワードを使えば一発で取得できる)。
// name()メソッドでルートに名前を付けられる。
// ルートの名前を定義すると、route関数でURLを生成できるようになる。
// ルートの名前に特定のルールや制約はないが、ドット区切りでリソースとアクションを表現するのが一般的。
Route::get('/folders/{id}/tasks/{task_id}/edit', [TaskController::class,"showEditForm"])->name('tasks.edit');

// これも一応ルーティング定義の例
Route::get('/folders/create', [FolderController::class,"showCreateForm"])->name('folders.create');
Route::post('/folders/create', [FolderController::class,"create"]);

// ルートパラメータの値が引数として渡される例。
public function showEditForm(int $id, int $task_id)
{
    //
}

// route関数でURLを生成する例。
// 基本：第一引数にルートの名前を指定すると、ルートのURLそのものが生成される。
$url = route('folders.create');
echo $url; // 'http://127.0.0.1:8000/folders/create'
// 応用：ルートのURLにルートパラメータの指定がある場合は、第二引数に連想配列で指定する。
$url = route('tasks.edit', [
    'id' => 11,
    'task_id' => 22,
]);
echo $url; // 'http://127.0.0.1:8000/folders/11/tasks/22/edit'


// リダイレクトをする際に使用するredirect関数にもrouteは使える(その時redirect関数には何も引数を入れない)。
return redirect()->route('tasks.index', [
    'id' => $id
]);

// ※redirect関数の基本的な一番使い方(固定URLにリダイレクト)。
return redirect('URL');


// Bladeテンプレート内でリンクを生成する場合は次のように使用する。
<a href="{{ route('folders.edit', ['id' => $folder->id]) }}">編集</a>

```

<a id="コントローラー"></a>
## コントローラー

### リソースコントローラ
(必ず使うというわけではないが、命名規則等を参考にしようかな。)
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class SampleController extends Controller
{
    // サイトのTOPページ、投稿一覧などを表示するためのメソッドを書く。
    public function index()
    {
        //
    }

    // 新規投稿作成ページを表示するためのメソッドを書く。
    public function create()
    {
        //
    }

    // 新規投稿をデータベースに保存するためのメソッドを書く。
    public function store(Request $request)
    {
        //
    }

    // 投稿の個別ページを表示するためのメソッドを書く。
    public function show(string $id)
    {
        //
    }

    // 投稿の編集ページを表示するメソッドを書く。
    public function edit(string $id)
    {
        //
    }

    // 編集した投稿をデータベースに上書き保存するメソッドを書く。
    public function update(Request $request, string $id)
    {
        //
    }

    // 投稿を削除するメソッドを書く。
    public function destroy(string $id)
    {
        //
    }
}
```






---------------------------------------------

































