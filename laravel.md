
# Laravel覚書

* 目次
    * [図解](#図解)
    * [参考サイト](#参考サイト)
    * [便利なVSCodeの拡張機能](#便利なVSCodeの拡張機能)
    * [インストール](#インストール)
    * [gitからcloneする時](#gitからcloneする時)
    * [サーバー起動](#サーバー起動)
    * [データベース等の設定](#データベース等の設定)
    * [XAMPPでMySQLが起動しない時の対処法](#XAMPPでMySQLが起動しない時の対処法)
    * [enum（列挙型）](#enum（列挙型）)
    * [マイグレーション](#マイグレーション)
    * [モデル](#モデル)
    * [シーダー](#シーダー)
    * [ファクトリー](#ファクトリー)
    * [ルーティング](#ルーティング)
    * [コントローラー](#コントローラー)
    * [リクエスト](#リクエスト)
    * [ビュー](#ビュー)
    * [Breeze](#Breeze)
    * [Breezejp](#Breezejp)
    * [TailwindCSS](#TailwindCSS)
    * [ユーザーアクションの認可](#ユーザーアクションの認可)



<a id="図解"></a>
## 図解
![図解](./laravel.drawio.svg)


<a id="参考サイト"></a>
## 参考サイト
[ベストプラクティス](https://github.com/alexeymezenin/laravel-best-practices/blob/master/japanese.md)

<a id="便利なVSCodeの拡張機能"></a>
## 便利なVSCodeの拡張機能

* PHP Intelephense
* laravel extension pack

<a id="インストール"></a>
## インストール

### カレントディレクトリにlaravelをインストールしてプロジェクトを自動生成するコマンド 
`composer create-project laravel/laravel [プロジェクト名]`  
(プロジェクトを作るたびにlaravelをインストールする必要がある)  
(基本的にはC:\xampp\htdocs内でコマンドを実行)  

<a id="gitからcloneする時"></a>
## gitからcloneする時

### 概要
git cloneしてきたLaravelプロジェクトにはvendorディレクトリと.envファイルが含まれていないので、作成する必要がある。
※vendorディレクトリと.envファイルは、通常Gitの管理下におかない(初期状態で.gitignoreに入っている)。

### vendorディレクトリを作る
下記のコマンドを実行する。  
`composer install`

### .envファイルを作る
.env.exampleファイルを下記のコマンドでコピーして.envファイルを作成する。  
`cp .env.example .env`

### アプリケーションキーを初期化する
.env.exampleをコピーして.envを作成してもアプリケーションキーは設定されていないので、下記コマンドで設定する。  
`php artisan key:generate`
#### ※アプリケーションキーとは？
.envファイルのAPP_KEY=の項目。暗号化された値を安全に扱うためのもの。



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

### config\database.php
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


<a id="enum（列挙型）"></a>
## enum（列挙型）

(仮)複数の定数をまとめて管理できる機能。


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
        // Schema::createメソッドの第一引数は作成するテーブルの名前
        // Schema::createメソッドの第二引数はテーブルの構造を定義するためのコールバック関数(無名関数)
        // そのコールバック関数は、Schema::createメソッドが内部で生成したBlueprintクラスのインスタンスを引数として受け取る(通常は$tableという変数名)
        // 無名関数内でそのインスタンスからインスタンスメソッドにアクセスし、カラムを作成する
        // $table->型名になっているインスタンスメソッド('カラム名');という書き方が基本。
        // 例外として、$table->datetimes();は、1行でcreated_atとupdated_atの2列を作成する。
        Schema::create('folders', function (Blueprint $table) {
            $table->id();
            $table->string('title', 20);
            $table->datetimes();
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

### Blueprintクラスのインスタンスメソッドのうち、よく使われるもの
```php
<?php
// Blueprintクラスには、テーブルやカラムを定義するためのさまざまなインスタンスメソッドが用意されている。

// カラム定義 整数型
$table->id(); // 自動増分ID。ショートカットメソッド。内部的には$table->bigIncrements('id')と同じ。
$table->bigInteger('column_name'); // 8バイトの整数カラム
$table->integer('column_name'); // 4バイトの整数カラム
$table->mediumInteger('column_name'); // 3バイトの整数カラム
$table->smallInteger('column_name'); // 2バイトの整数カラム
$table->tinyInteger('column_name'); // 1バイトの整数カラム
$table->unsignedBigInteger('column_name'); // 符号なしの8バイト整数カラム
$table->unsignedInteger('column_name'); // 符号なしの4バイト整数カラム

// カラム定義 文字列型
$table->string('column_name', 255); // 255文字までの文字列カラム
$table->text('column_name'); // 通常のテキストデータ。最大おおよそ2万文字
$table->mediumText('column_name'); // 大きめのテキストデータ。最大おおよそ500万文字
$table->longText('column_name'); // 非常に大きなテキストデータ。最大おおよそ14億文字

// カラム定義 日時型
$table->date('column_name'); // 日付カラム
$table->dateTime('column_name', $precision = 0); // 日時カラム
$table->time('column_name', $precision = 0); // 時間カラム
$table->timestamp('column_name', $precision = 0); // タイムスタンプカラム
$table->timestamps($precision = 0); // created_at と updated_at カラムを追加
$table->softDeletes($precision = 0); // deleted_at カラムを追加

// カラム定義 ブール型
$table->boolean('column_name'); // ブールカラム

// カラム定義 その他
$table->binary('column_name'); // バイナリデータカラム
$table->float('column_name', $total = 8, $places = 2); // 浮動小数点カラム
$table->decimal('column_name', $total = 8, $places = 2); // 固定小数点カラム
$table->json('column_name'); // JSONデータカラム
$table->jsonb('column_name'); // JSONBデータカラム

// インデックスを定義するメソッド
$table->primary('column_name'); // 主キーを定義
$table->unique('column_name'); // ユニークインデックスを定義
$table->index('column_name'); // 標準インデックスを定義

// 外部キーを定義するメソッド
// 子テーブルのfolder_idカラムが、常にfoldersテーブルの有効なidを参照することを強制
// 子テーブルにある各レコードのfolder_idは、必ずfoldersテーブルに存在するidでなければならなくなる。
$table->foreign('folder_id')->references('id')->on('folders');

// その他の便利なメソッド
$table->increments('column_name'); // 自動増分の主キーを定義（整数）
$table->bigIncrements('column_name'); // 自動増分の主キーを定義（大きな整数）
$table->rememberToken(); // remember_tokenカラムを追加（Laravel認証用）
$table->nullableMorphs('morphable'); // typeとidカラムを追加（多形関係用）
$table->uuid('column_name'); // UUIDカラムを追加

// カラムの属性を変更するメソッド
$table->nullable(); // カラムをNULL許可にする
$table->default($value); // デフォルト値を設定する
```

### マイグレーションの実行コマンド  
`php artisan migrate`

### 全てのテーブルを削除し、全てのマイグレーションを再実行するコマンド  
`php artisan migrate:fresh`



<a id="モデル"></a>
## モデル

### 前提
Laravelが提供するデータベース操作方法は以下の3つ。

* DBクラスと素のSQL
    * テーブルに対して直接クエリを実行する。
* DBクラスのクエリビルダ
    * SQLの内容をPHPライクに記述し、内部でLaravelがクエリを作成する。
    * テーブルに対して直接クエリを実行する。
* Eloquent ORM
    * Modelクラスを定義する必要がある。
    * クエリビルダの全メソッドに加え、Eloquentの独自メソッドを使用できる。
    * リレーションの定義、タイムスタンプの自動更新、アクセサとミューテタ等の便利機能がある。

ここでは、主要な方法であるEloquent ORMの使用時に定義するModelクラスについて説明する。

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

class Folder extends Model
{
    use HasFactory;

    // テーブル名を明示的に指定する
    protected $table = 'folders';
}
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

### VS Codeでメソッド補完
VSCodeの拡張機能「PHP Intelephense」によって、モデルの基本メソッドが補完される。

以下のコマンドを実行することで、自作モデルのメソッドも補完される  
`composer require --dev barryvdh/laravel-ide-helper`  
`php artisan clear-compiled`  
`php artisan ide-helper:generate`  
`php artisan ide-helper:models --nowrite`  

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


// テーブルのレコード数を取得する。戻り値は整数値(int)。
ModelClass::count();

// 新しいインスタンス(レコード)の作成、追加、保存を簡潔に書く方法。
// 'created_at'(作成日)と'updated_at'(更新日)は自動的に追加される。
ModelClass::create([
    'folder_id' => 1,
    'title' => "サンプルタスク {$num}",
]);
// createの別バリエーション
ModelClass::firstOrCreat([]); // レコードが存在しなければ作成
ModelClass::updateOrCreate([]); // 存在するレコードを更新し、存在しない場合は作成

// 指定したIDのレコードを削除
ModelClass::destroy(1);
ModelClass::destroy([1, 2, 3]);
```

### クエリビルダーメソッド
ドキュメント：[Laravel 11.x データベース：クエリビルダ](https://readouble.com/laravel/11.x/ja/queries.html)



```php
<?php
// WHERE句
where
orWhere
whereNot
whereAny
whereAll

whereJsonContains
whereJsonLength

whereBetween / orWhereBetween
whereNotBetween / orWhereNotBetween
whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns
whereIn / whereNotIn / orWhereIn / orWhereNotIn
whereNull / whereNotNull / orWhereNull / orWhereNotNull
whereDate / whereMonth / whereDay / whereYear / whereTime
whereColumn / orWhereColumn
whereExists
whereFullTextとorWhereFullText

論理グループ化


// 順序
orderBy
latest / oldest
inRandomOrder
reorder

// グループ化
groupBy / having
havingBetween
havingRaw
```




```php
<?php
// 戻り値はクエリビルダーインスタンス（具体的にはIlluminate\Database\Eloquent\Builderのインスタンス）。
// クエリビルダーメソッドはModelClassからも$queryBuilderInstanceからも呼べる。

// クエリビルダーインスタンスを呼び出して返すだけのメソッド。
// 可読性を高めるために使う
ModelClass::query()

// ※query()の使用例
$users = User::query()
    ->select('name', 'email')
    ->whereIn('address', request('address'))
    ->oldest()
    ->take(10)
    ->toArray()

// 条件を指定してフィルタをかける
// 比較演算子は文字列で、'=', '>', '<', '>=', '<=', '!=', 'LIKE', 'IN'などを指定。
// 比較演算子の省略は'='の使用と同義。
ModelClass::where('カラム名', '比較演算子', '比較する値')
$queryBuilderInstance->where('カラム名', '比較演算子', '比較する値')

// 複数の条件を指定してフィルタをかける
ModelClass::where([
    ['カラム名1', '比較演算子1', '比較する値1'],
    ['カラム名2', '比較演算子2', '比較する値2'],
    // 他の条件...
])

// 条件：指定した値のリストのいずれかに一致するレコード。
ModelClass::whereIn('カラム名', [1, 2, 3])
// 条件：指定した値のリストに含まれないレコード。
ModelClass::whereNotIn('カラム名', [1, 2, 3])

// 条件：カラムがNULLのレコード。
ModelClass::whereNull('カラム名')
// 条件：カラムがNULLでないレコード。
ModelClass::whereNotNull('カラム名')

// 特定のカラムのみを取得するために使用。
ModelClass::select('カラム名1', 'カラム名2')
// where()の条件が満たされない場合に適用される条件を追加
ModelClass::where('カラム名', '比較演算子', '比較する値')->orWhere('カラム名', '比較演算子', '比較する値')

// 日付単位の条件を指定してフィルタをかける。
// whereDateはwhereの日付特化版。
// ※この場合の条件は、dateカラムが今日の日付のレコード。
ModelClass::whereDate('date', Carbon::now())

// 指定されたカラムでレコードを並び替える。
ModelClass::orderBy('カラム名', 'ascまたはdesc')
```
### クエリ実行メソッド
```php
<?php
// クエリビルダーインスタンスからクエリを実行して、実際の結果を取得するために使用する。

$queryBuilderInstance->get(); // 全てのレコードを取得。戻り値はCollectionクラス。
$queryBuilderInstance->first(); // 最初のレコードを取得。戻り値はModelインスタンス。無いときはnull
$queryBuilderInstance->count(); // レコードの件数を取得。
$queryBuilderInstance->exists(); // レコードが存在するかどうかをチェック。戻り値は真偽値(bool)。
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


<a id="ファクトリー"></a>
## ファクトリー

参考サイト：[ファクトリーの使い方](https://office54.net/iot/laravel/factory-test-data-create)

Factoryを使うと、Seederよりも簡単に大量のテストデータを生成できる。

### ファクトリーファイルの新規作成コマンド  
`php artisan make:factory モデル名Factory`  
例：  
`php artisan make:factory FolderFactory`  
※database/factories内にFolderFactory.phpというファクトリーファイルが作成される。

### ファクトリーファイルの解説
```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

// Factoryクラスを継承している
class TaskFactory extends Factory
{
    // 戻り値は配列(レコード)
    public function definition(): array
    {
        return [
            // どのようなデータを生成するかをここに定義する。
            'title'=>fake()->text(15),
            'due_date'=>fake()->dateTime(),
            'status'=>fake()->numberBetween(1, 2),
            // Folderモデルへのリレーションを定義
            'folder_id'=>\App\Models\Folder::factory(),
        ];
    }
}
```
### fake()メソッド
fake()メソッドは、Laravelが提供するFakerライブラリを使って、さまざまな種類のダミーデータを生成するために使用される。  
※Fakerは、名前、住所、電話番号、テキスト、日付などのリアルなダミーデータを生成するライブラリ。

利用できるメソッド一覧
```php
<?php
fake()->text($maxNumOfChara) // テキスト（日本語非対応）
fake()->realText() // テキスト（日本語対応）
fake()->word() // 単語
fake()->paragraph() // 複数の文章
fake()->address() // 住所
fake()->country() // 国
fake()->postcode() // 郵便番号
fake()->prefecture() // 都道府県
fake()->city() // 市
fake()->company() // 会社名
fake()->phoneNumber() // 電話番号
fake()->email() // メールアドレス
fake()->name() // 人名
fake()->lastName() // 性
fake()->firstName() // 名
fake()->numberBetween($min=〇, $max=〇)	// 指定した範囲内の数値
fake()->date() // 年月日
fake()->dateTime() // 年月日　時分秒
fake()->dateTimeBetween($startDate=〇, $endDate=〇) // 'now' '+2 week' '-1 years'
fake()->year() // 年
fake()->month() // 月
fake()->dayOfMonth() // 日
fake()->time() // 時分秒
```

### ファクトリーの実行
```php
<?php
// DatabaseSeeder.php
// シーダーで利用されるファイルだが、ファクトリーでも利用される。
// そのため、実行コマンドはシーダーと同じphp artisan db:seed

namespace Database\Seeders;

use App\Models\User;
use App\Models\Folder;
use App\Models\Task;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);

        // ファクトリーを使ってテストデータを100個作成。
        Task::factory(100)->create();
        // ※ModelClass::factory()はEloquentモデルのクラスメソッドではなく、Laravelが提供するマジックメソッド。
        // ※このメソッドはHasFactoryトレイトを使用して実現され、各モデルに対して対応するファクトリクラスを動的に解決する。

        // $this->call([
        //     FoldersTableSeeder::class,
        //     TasksTableSeeder::class,
        // ]);
    }
}
```


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
Route::get('/folders/{id}/tasks/{task_id}/edit', [TaskController::class,"edit"])->name('tasks.edit');

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

### 参考サイト
[Laravel 11.x コントローラ](https://readouble.com/laravel/11.x/ja/controllers.html)


### 基本
* コントローラーの役割
    1. リクエストを受け取る
    1. Modelへの処理指示
    1. Viewを表示

※※重要！※※  
**コントローラーは指示役に徹する。自分自身で一切処理は行わない。**

作成コマンド  
`php artisan make:controller SampleController`  
アプリケーションのすべてのコントローラは、デフォルトでapp/Http/Controllersディレクトリへ設置される。


#### Viewを表示
view関数を使う
```php
<?php
// 第一引数に、表示したいビューの名前を指定。
// 通常、resources/viewsディレクトリ配下からのパス(「.blade.php」は不要)。
return view('folders/create');

// ビューにデータを渡す場合は、第二引数に連想配列として指定。
// 渡したデータはビュー内で使用され、HTMLを動的に生成するために利用されます。
return view('folders/edit', [
    'folder_id' => $folder->id,
    'folder_title' => $folder->title,
]);

// ビューの変数とコントローラで定義した変数の名前が一致している場合は、compact関数を利用してスッキリ書くこともできる。
// ※compact関数とは、変数名とその値から配列を作成するPHPの関数。
// compact関数使用例
return view('tasks/edit', compact('hoge', 'fuga', 'piyo'));
// ※以下と同義
return view('tasks/edit', [
    'hoge' => $hoge,
    'fuga' => $fuga,
    'piyo' => $piyo,
]);

```


### リソースコントローラ
作成コマンド  
`php artisan make:controller SampleController  --resource`  
(必ず使うというわけではないが、命名規則等を参考にしようかな。)
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

// $idをキーとして使うなら、とりあえず整数型(int)にしておく。
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

    // あんまり使わないかも？
    // 投稿の個別ページを表示するためのメソッドを書く。
    public function show(int $id)
    {
        //
    }

    // 投稿の編集ページを表示するメソッドを書く。
    public function edit(int $id)
    {
        //
    }

    // 編集した投稿をデータベースに上書き保存するメソッドを書く。
    public function update(Request $request, int $id)
    {
        //
    }

    // ※自作
    // 投稿の削除ページを表示するメソッドを書く。
    public function showDestroy(int $id)
    {
        //
    }

    // 投稿を削除するメソッドを書く。
    public function destroy(int $id)
    {
        //
    }
}
```
### コントローラーミドルウェア
ルーターとコントローラーの間でログイン処理等を実装できるもの。

ミドルウェアはルートファイルの中で、コントローラのルートに対して指定する。※他の方法もある。
```php
<?php

Route::get('profile', [UserController::class, 'show'])->middleware(['auth', 'verified'])->name('profile.show');
// 'auth'だの'verified'だのは、「bootstrap\app.php」と「Illuminate\Foundation\Configuration\Middleware;」を参考。
```


<a id="リクエスト"></a>
## リクエスト

### Request
LaravelはHTTPリクエストが来ると、このリクエストの情報をIlluminate\Http\Requestクラスのインスタンスにラップし、コントローラーメソッドの引数として依存性注入する。

例：  
```php
<?php
// ルートに紐づいたコントローラに対し、クラス名がタイプヒントされた引数を指定すると、そのクラスのインスタンスを自動的に生成し、引数として渡してくれるようになる。
// Laravelに組み込まれたサービスコンテナが担う機能の一つで、 依存性注入と言う。
// サービスコンテナとは、 便利な「自動インスタンス化マシン」のようなもの。

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class SampleController extends Controller
{
    // 「Request $request」の部分が依存性注入
    public function store(Request $request)
    {
        //
    }
}
```
「Request $request」は、HTTPリクエストに関するすべての情報を持つオブジェクト。  
ユーザーが送信したデータ、リクエストメソッド、ファイル、ヘッダー情報などを取得、操作するために使用される。  
Requestオブジェクトを使うことで、リクエストに関する情報を簡単かつ効率的に扱うことができる。  
例：  
```php
<?php
public function store(Request $request)
{
    $folder = new Folder();

    // ※$request->のあとにある変数名は、HTMLのタグのname属性で付けた名前。
    $folder->title = $request->title;

    $folder->save();
    return redirect()->route('tasks.index', [
        'id' => $folder->id,
    ]);
}
```

### FormRequest
FormRequestクラスはIlluminate\Http\Requestクラスを継承している。    
そのFormRequestクラスを継承したクラスを利用することで、簡単に認可・バリデーションを行える。    
コントローラーからバリデーション処理を完全に切り離し、Fat Controller化を防ぐ効果もある。  

FormRequestの子クラスを作成するコマンド  
`php artisan make:request [任意のクラス名]`  
例：  
`php artisan make:request SampleRequest`  
これにより、app/Http/Requests/SampleRequest.phpが作成されます。  

作成されるコードの例:  
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;


class SampleRequest extends FormRequest
{
    // リクエストが認可されるかどうかを決定するためのメソッド。
    // 戻り値はbool
    // falseが返った場合、バリデーションすら行わず403 Forbiddenを返しリクエストが拒否される。
    // trueが返った場合はバリデーションに処理が進む。つまり認可・許可されたことになる。
    // 条件によってtrueにしたり、falseにしたりと、便利な使い方もできる。
    public function authorize(): bool
    {
        return true;
    }

    // バリデーションルールを配列として定義し、返すメソッド。
    // 定義したルールに基づいてバリデーションが実行される。
    // バリデーションが通らなければ、直前の画面にリダイレクトされる。
    // バリデーションが通れば、コントローラー内部へと処理が移る。
    public function rules(): array
    {
        return [
            // name：必須入力で最大20文字
            'name'  => 'required|max:20',
            // tel：入力任意、入力された時はハイフン区切りの電話番号の形式で
            'tel'   => 'nullable|regex:/^[0-9]{2,4}-[0-9]{2,4}-[0-9]{3,4}$/',
            // email：必須入力でメールアドレスの形式
            'email' => 'required|email',
            // body：必須入力で最大1000文字まで
            'body'  => 'required|max:1000',
            // date：値を日付形式に指定
            // after_or_equal：特定の日付（この場合はtoday）以前の日付の入力を不可に（制限）する
            'due_date' => 'required|date|after_or_equal:today',
        ];
    }

}
```

FormRequestの子クラスを利用するのはとても簡単。  
コントローラの使いたいメソッドに、Requestの代わりに依存性注入するだけ。  

例：  
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
// SampleRequestを読み込む。
use app\Http\Requests\SampleRequest;

class SampleController extends Controller
{
    // 「SampleRequest $request」の部分が依存性注入
    public function store(SampleRequest $request)
    {
        //
    }
}
```
たったのこれだけで、コントローラのメソッド（ここではstore()）が呼び出される前にauthorize()、通った場合はrules()の処理が実行される。  


#### 追加処理
```php
<?php
// バリデーション前に処理を行う
// prepareForValidationメソッドをオーバーライド
protected function prepareForValidation()
{
    //
}

// バリデーション成功時に処理を行う
// passedValidation メソッドを追加
protected function passedValidation()
{
    //
}

// バリデーション失敗時に処理を行う
// failedValidationメソッドをオーバーライド
protected function failedValidation(Validator $validator)
{
    //
}

// 追加バリデーションの実行
// 方法の一つとしてwithValidatorメソッドが用意されている。
public function withValidator(Validator $validator): void
{
    //
}
```

#### エラーメッセージの日本語化＆カスタマイズ
* 2つの方法がある。
    1. Laravelの言語ファイルを作成＆設定する。
    1. FormRequestの子クラスで定義する(messagesメソッドをオーバーライドする)。

#### 参考サイト
[LaravelのFormRequestをちゃんと理解する](https://laranote.jp/understanding-laravel-formrequest/)


<a id="ビュー"></a>
## ビュー


### 参考サイト
[Laravel 11.x Bladeテンプレート](https://readouble.com/laravel/11.x/ja/blade.html)


### asset関数
publicディレクトリ配下へのURLを生成する。
例：  
```php
<?php

asset('css/styles.css')

// ビューの中で呼び出す場合は、{{ }}をつける。
{{ asset('css/styles.css') }}
```



### レイアウト

#### コンポーネント
独立したUI要素を管理したり、複数のビューで再利用する場合に適している。  
※ユーザー認証システムのBreezeでも使う。

#### テンプレート継承
各ページが同じレイアウトや構造を共有し、一部だけを変更する場合に適している。  
※コンポーネントの導入前にアプリケーションを構築するための主な方法だった。

```php
<?php
// 親ビューに設定
// 子ビューにsectionで定義したデータを表示する
@yield('セクション名')

// 子ビューに設定
// 親ビューを継承する（読み込む）
@extends('親ビュー名')
// sectionでデータを定義する
// 読み込んだ親ビューの@yield('セクション名')部分にはめ込む用。
// ※同じセクション名の@yield()部分に表示される。
// パターン1
@section('セクション名', '文字列')
// パターン2
@section('セクション名')
    <h1>Welcome to the Home Page</h1>
    <p>This is the home page content.</p>
@endsection

// 別のビュー内からBladeビューを読み込む。
@include('読み込むビュー名')

```
* @extends()、@include()で読み込むビュー名について。
    1. resources/viewsディレクトリ配下からのパス。
    1. .blade.phpの拡張子は省略することができる。
    1. ディレクトリの階層をドットで区切った形式でビュー名を指定。


### CSRFフィールド、Methodフィールド
* CSRFフィールド：formの内部に@csrfと記述するだけでCSRF攻撃を防げる。  
* Methodフィールド：HTMLフォームで許可されているのはGETリクエスト(データ取得)とPOSTリクエスト(データ送信)のみのため、フォームのPUT, PATCH, DELETE, OPTIONSリクエストを使う場合はPOST送信のフォームの中で@method関数を使用し、擬似的にリクエストを実現させる。

例：  
```php
<?php
<form action="{{ route('hoges.destroy', ['id' => $id]) }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">削除</button>
</form>
```




<a id="Breeze"></a>
## Breeze

### カレントディレクトリにLaravel Breezeをインストールするコマンド 
`composer require laravel/breeze --dev`  
`php artisan breeze:install`  


#### ※質問：Which testing framework do you prefer?
テストに使用するフレームワークを選択する

* Pest (シンプル。初心者にも優しい。)
* PHPUnit (多機能。習得には多少の時間がかかる。複雑なテストや高度なカスタマイズが必要な場合。)


### gitからcloneする時

#### 概要
node_modulesディレクトリは通常Gitの管理下におかないため、リポジトリをクローンした後に再度インストールする必要がある。  
※node_modulesにはプロジェクトに必要な依存関係がすべて含まれており、これがないとフロントエンドアセットがコンパイルできず、正しく機能しない。

#### node_modulesディレクトリをインストールする方法
以下のコマンドを実行して、必要なnodeモジュールをすべてインストールする。  
`npm install`  

フロントエンドアセットをコンパイルする（オプションだが、通常は必要）  
`npm run dev`  
※本番環境の場合  
`npm run production`  
  
これらの手順を実行すると、Laravel Breezeアプリケーションが実行できるようになる。  



### 特定のルートグループにミドルウェアを適用する方法
```php
<?php
// ※ミドルウェアは、リクエストがコントローラーやルートに到達する前に実行されるフィルターのようなもの。
// ユーザーがページにアクセスする前に、認証の有無などがチェックされる。

// Route::middleware()メソッドを使用し、ルートグループにguestミドルウェアを適用。
// ->group(function () { ... })メソッドは複数のルートをグループ化する。
// ※group()メソッドの引数は無名関数で、その中に個々のルート定義を記述する。
// ※グループ内のすべてのルートに対して、共通の設定が適用される。
Route::middleware('guest')->group(function () {
    // 認証されていないユーザーのみがアクセスできるルート
    
    Route::get('register', [RegisteredUserController::class, 'create'])
                ->name('register');

    Route::post('register', [RegisteredUserController::class, 'store']);
});

// Route::middleware()メソッドを使用し、ルートグループにauthミドルウェアを適用。
Route::middleware('auth')->group(function () {
    // 認証されているユーザーのみがアクセスできるルート

    Route::get('verify-email', EmailVerificationPromptController::class)
                ->name('verification.notice');
});
```
### 単一のルートにミドルウェアを適用する場合
```php
<?php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```




<a id="Breezejp"></a>
## Breezejp

Breezejpをインストールするコマンド  
`composer require askdkc/breezejp --dev`  
必要な言語ファイルの出力を実行するコマンド  
`php artisan breezejp`

### (仮)lang
lang/作成コマンド
`php artisan lang:publish`  

resources/lang/en  
resources/lang/ja  


<a id="TailwindCSS"></a>
## TailwindCSS

### 基本

```html
<head>
    <!-- スタイルを適用したいbladeファイル（レイアウトを共通化するためのファイル）に以下を追記。 -->
    @vite('resources/css/app.css')
</head>
```

```css
@tailwind base;
@tailwind components;

/* 独自クラス（コンポーネント）を作成することもできる。 */
.my-bgc{
    @apply bg-green-300
}

@tailwind utilities;
```

### VSCodeの便利な拡張機能

|||
|-|-|
|Tailwind CSS IntelliSense|コードのutility classにカーソルを当てるとCSSの設定を確認することができる。|
|PostCSS Language Support|エディタが@tailwindを正しく認識してくれるようになり、「Unknown at rule @tailwind」という警告と黄色い波線が消える。|

### 参考サイト
* [Tailwind CSS Cheat Sheet](https://tailwindcomponents.com/cheatsheet/)
* [Tailwind CSSで独自クラス（コンポーネント）を作成する](https://zenn.dev/takashi5816/articles/7d9d14a17a3ec0)
* [初めてでもわかるTailwind CSS入門 基礎編](https://reffect.co.jp/html/tailwindcss-for-beginners)
* [Vite + Tailwind CSS + Laravel Breezeを使った開発環境の構築](https://zenn.dev/nenenemo/articles/46d43854cd01c5)




<a id="ユーザーアクションの認可"></a>
## ユーザーアクションの認可

### 参考サイト
[Laravel 11.x 認可](https://readouble.com/laravel/11.x/ja/authorization.html)

### 基本
主要な方法として、ゲートとポリシーがある。
* ゲートは単独で制限をかけたいときに使う。
* ポリシーはモデルごと、リソースコントローラーごとなど、まとめて制限をかけたいときに使う。
2つの方法を組み合わせながら、ユーザー単位での情報アクセス制限を柔軟に実現することができる。

※おまけ：ミドルウェアという手段もあり、ルート設定の段階で制限をかけたいときに使う。


### ゲート
Gate::defineメソッド  
```php
<?php
// app/Providers/AuthServiceProvider.php

use Illuminate\Support\Facades\Gate;
use App\Models\User;
use App\Models\Post;

public function boot(): void
{
    $this->registerPolicies();

    // Gate::defineメソッド
    // 特定のアクションに対する認可ロジックを定義するために使用される。
    // 通常、AuthServiceProviderのbootメソッド内で定義される。
    // 第一引数は文字列で、定義する認可アクションの名前を指定する。
    // 認可アクションの名前について↓
    // 一意である必要がある。
    // 後でGate::allowsやGate::deniesメソッドを使って認可チェックを行うときに使用される。
    // 投稿の閲覧権限を定義するなら'view-post'、投稿の編集権限を定義するなら'edit-post'といったように、説明的でわかりやすい名前を付ける。
    Gate::define('view-post', function (User $user, Post $post) {
        // 認可ロジック
        return $user->id === $post->user_id; // trueまたはfalse
    });
}
```
Gate::allowsメソッド  
Gate::deniesメソッド  
```php
<?php

public function show(Post $post)
{
    // Gate::allowsメソッド  
    // 認証されているユーザーが特定のアクションを実行できるかどうかを確認するために使用される。
    // 第一引数：アクション名
    // 第二引数：モデルインスタンス
    // 認可アクションが許可されている場合にtrueを返す。
    // ※Gate::deniesメソッドは逆に、認可アクションが拒否されている場合にtrueを返す。
    // 認可が必要なアクションを実行する前に、コントローラ内でゲート認可メソッドを呼び出すのが一般的
    if (Gate::allows('view-post', $post)) {
        // 認可されている場合の処理
        return view('posts.show', compact('post'));
    } else {
        // 認可されていない場合の処理
        abort(403, 'Unauthorized action.');
    }
}
```


### ポリシー

#### ポリシーの生成
* コマンドを使用してポリシーを生成できる。
* 生成するポリシーはapp/Policiesディレクトリへ配置される。
* ※app/Policiesディレクトリが存在しない場合、Laravelが作成する。
* 命名例：Postモデルは、PostPolicyポリシークラスに対応する。

生成コマンド
`php artisan make:policy PostPolicy --model=Post`

#### ポリシーの登録
* 作成したPolicyをAuthServiceProviderに登録する。  
* Gate::policyメソッドを使用し、AppServiceProviderのbootメソッド内で、ポリシーと対応するモデルを登録できる。

```php
<?php

use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * アプリケーションの全サービスの初期起動処理
 */
public function boot(): void
{
    Gate::policy(Post::class, PostPolicy::class);
}
```

#### ポリシーの作成

* ポリシークラスを登録したら、認可するアクションごとにメソッドを追加できる。
* メソッドは、権限があるかどうかを示すbool値を返す必要がある。
* ポリシーが認可するさまざまなアクションの必要に合わせ、ポリシーに追加のメソッドをどんどん定義できる。
* ポリシーメソッドには任意の名前を付けることができる。
* ポリシーを生成するときに--modelオプションを使用した場合、はじめからviewAny、view、create、update、delete、restore、forceDeleteアクションのメソッドが用意される。

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 指定した投稿をユーザーが更新可能かを判定
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

#### モデルのないメソッド
一部のポリシーメソッドは、現在認証済みユーザーのインスタンスのみを受け取る。  
この状況は、createアクションを認可するばあいに頻繁に見かける。  
たとえば、ブログを作成している場合、ユーザーが投稿の作成を認可されているかを確認したい場合がある。  
このような状況では、ポリシーメソッドはユーザーインスタンスのみを受け取る必要がある。  

```php
<?php
/**
 * 指定ユーザーが投稿を作成可能か確認
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```















---------------------------------------------



