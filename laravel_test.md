# Laravel 覚書

- 目次
  - [概要](#概要)
  - [種類](#種類)
  - [AAA](#AAA)



<a id="概要"></a>

## 概要
[Laravel 12.x HTTPテスト ドキュメント](https://readouble.com/laravel/12.x/ja/http-tests.html)

**人間が手動でブラウザを開いて、ログインして、画面の数字を指差し確認する」というような作業を、PHPのコードで記述してロボットにやらせているだけ**  
結局 『ログインして、ボタン（URL）を押して、結果を見る』 というような人間の動きをなぞっているだけ

### $this
テストクラスのインスタンス$thisは、親クラスであるTestCaseの機能と性質を受け継いでいる
#### 機能
様々なテスト用メソッドなど
#### 性質
$thisは中にこういうのを持ってる
* アプリケーション（Laravel本体）
* 認証状態（actingAsで変更される）
* セッション
* HTTPクライアント的な機能（get/post）
* DB接続
* 各種テストヘルパ
つまり「Laravelの世界」が丸ごと入ってる

※ メソッドごとに新しいインスタンスが作られる  
※ つまり$thisはメソッドごとに独立した一つの世界  
※ メソッドが10個あれば、基本的に10回インスタンス化される  

### Pest
PestはPHPUnitの上に乗ってるラッパー、という位置づけ。
#### イメージ
Pest（書きやすい書き方）  
   ↓  
PHPUnit（実行エンジン）  
   ↓  
PHP（実行）  


### イントロダクション
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本のテスト例
     */
    public function test_the_application_returns_a_successful_response(): void
    {
        $admin = User::factory()->create();
        $this->actingAs($admin);
        $response = $this->get('admin/');
        $response->assertOk();
    }
}
```

### ルートのキャッシュ
アプリケーションに多くのルートファイルがある場合、テストケースにIlluminate\Foundation\Testing\WithCachedRoutesトレイトを追加すると、キャッシュを利用して効率をよくできる。




<a id="種類"></a>

## 種類
テストには二種類ある。

- Feature テスト
  - Laravel 用に拡張された TestCase クラスを継承している
  - use Tests\TestCase;
  - Laravel の機能がテスト内で全て使える。
  - 実行速度が遅い。
- Unit テスト
  - 素の PHPUnit の TestCase クラスを継承している
  - use PHPUnit\Framework\TestCase;
  - Laravel の機能が使えず、素の PHP コードとしてのロジックテストを書くものになっている。
  - 実行速度が早い。
  - Laravel に依存しない形で書けるテストは極力 Unit テストにするべき。

実行コマンド  
`php artisan test`  
`php artisan test --filter=UserFeatureTest`  
`php artisan test --filter=UserFeatureTest::test_can_create_user`

こういうコマンドも(設定グループのみテスト実行)  
`php artisan test --group my-tests`
```php
use PHPUnit\Framework\Attributes\Group;

// 設定方法
#[Group('my-tests')]
class ApplicantDataTest extends TestCase
{
    // ...
}
```

### Feature テスト
作成コマンド。tests/Feature ディレクトリへ配置される。  
`php artisan make:test UserTest`

メソッド名は、test_から始まり、テストの内容がわかる名前にする（スネークケース推奨）。  
例：public function test_user_can_submit_form()


### Unit テスト
作成コマンド。tests/Unit ディレクトリへ配置される。  
`php artisan make:test UserTest --unit`



<a id="AAA"></a>

## AAA
Laravelテストにおける定番パターン

状況を準備(Arrange):ユーザー・DB・認証
↓
行動(Act):HTTP・メソッド実行
↓
結果を検証(Assert):レスポンス・DB・副作用（イベント・メール・通知・キューなど）

* 黄金パターン
    * 誰かが（actingAs）
    * 何かをし（get/post/put/delete）
    * responseが返る
    * responseにassert繋げて検問の連鎖
    * assertには「こうなるはずだ」と予言を書く。
        * 予言通りならPass（合格）で静かに進む。
        * 予言が外れたらFail（不合格）で止まって理由を教えてくれる。

### Arrange
**前提条件（インプット）が違うなら、メソッドを分ける**
$this->actingAs($user) は、「ログイン済みユーザー」をセットする。  
セットしない場合はゲスト（未ログイン）状態。  
Act工程で、「ログイン済みユーザー」としてミドルウェア（auth）を通過できるようになる。  

Arrange工程の主要メソッド・手段まとめ:

#### 認証・ユーザー

```php
// $thisを返す
$this->actingAs($user)           // 一般ユーザーとして認証
$this->actingAs($user, 'admin')  // ガード指定で認証
```

---

#### Factory・データ準備

```php
// 基本
User::factory()->create()               // DB保存あり
Post::factory()->count(3)->create()     // 複数件作成
User::factory()->make()                 // DB保存なし（単体テスト向け）

// state 指定
User::factory()->admin()->create()
Post::factory()->published()->create()

// リレーション付き
User::factory()
    ->hasPosts(3)
    ->create()
```

---

#### DB 状態管理

> `use` はクラス上部に記述

```php
use RefreshDatabase;      // 毎テスト後に DB をロールバック（推奨）
use DatabaseTransactions; // トランザクションで包んでロールバック

$this->seed()                    // DatabaseSeeder を実行
$this->seed(UserSeeder::class)   // 特定 Seeder を実行
```

---

#### リクエスト環境の準備

```php
$this->withSession(['key' => 'val'])
$this->withHeaders(['X-Foo' => 'bar'])
$this->withHeaders(['Accept' => 'application/json'])  // JSON レスポンスを期待
$this->withCookie('name', 'value')
$this->withCookies([...])
```

---

#### Fake・副作用の遮断

> act より**前**に呼ぶ

```php
Event::fake()        // イベント発火を遮断
Mail::fake()         // メール送信を遮断
Notification::fake() // 通知を遮断
Queue::fake()        // ジョブのキュー投入を遮断
Storage::fake('s3')  // ファイルストレージを遮断
Http::fake([...])    // 外部 HTTP 呼び出しを遮断
```

---

#### 時刻操作

```php
$this->travelTo(now()->addDays(3)) // 指定日時に移動
$this->travel(1)->days()           // 相対移動
$this->freezeTime()                // 現在時刻を固定
```

---

#### 設定・挙動制御

```php
$this->withoutExceptionHandling()           // 例外をそのまま投げる（デバッグ用）
config(['app.debug' => true])               // テスト内で設定を上書き
Gate::define('update-post', fn() => true)  // 認可ロジックをスタブ化
route('posts.store')                        // 名前付きルートで URL 生成
```


### Act
#### リクエストの作成
* アプリケーションにリクエストを送信するには、テスト内でget、post、put、deleteメソッドを呼び出す。
* これらのメソッドは、Illuminate\Testing\TestResponseのインスタンスを返す。
* そのインスタンスはレスポンスの検査を可能にするさまざまな有用なアサートを提供している。
* 通常、各テストはアプリケーションに対して 1 つのリクエストしか行わないようにする。１つのテストメソッドの中で複数のリクエストを実行すると、 予期せぬ動作が起きる可能性があるため。

リクエストメソッド
```php
// 指定したurlに対してGETリクエストを送る。
$response = $this->get($url)
// 指定したurlに対してPOSTリクエストを送る。
$response = $this->post($url, $data = [])
// 指定したurlに対してPUTリクエストを送る。
$response = $this->put($url, $data = [])
// 指定したurlに対してDELETEリクエストを送る。
$response = $this->delete($url)
```

#### ファイルアップロードのテスト
Illuminate\Http\UploadedFileクラスは、テスト用のダミーファイルまたは画像を生成するために使用できるfakeメソッドを提供している。  
これをStorageファサードのfakeメソッドと組み合わせると、ファイルアップロードのテストが大幅に簡素化される。  
たとえば、次の２つの機能を組み合わせて、アバターのアップロードフォームを簡単にテストできる。  



### Assert
#### Illuminate\Testing\TestResponseクラスが提供するアサート
※get、post、put、deleteテストメソッドが返すレスポンスからアクセスできる。

##### レスポンスのアサート
###### 基本
[一覧](https://readouble.com/laravel/12.x/ja/http-tests.html#assert-ok:~:text=%E5%8F%AF%E8%83%BD%E3%81%AA%E3%82%A2%E3%82%B5%E3%83%BC%E3%83%88-,%E3%83%AC%E3%82%B9%E3%83%9D%E3%83%B3%E3%82%B9%E3%81%AE%E3%82%A2%E3%82%B5%E3%83%BC%E3%83%88,-Laravel%E3%81%AEIlluminate)

###### Inertia固有
```php
$response->assertInertia(fn (Assert $page) => ...)

// Inertia固有のチェック
// React（フロントエンド）に正しくバトンを渡せているかを検証する
// これらは $response->assertInertia(fn (Assert $page) => ...) の中で使う。

// 指定したReactコンポーネント（例: Admin/User/Index）が呼び出されているか。
$page->component('ComponentName')
// React側にその変数が存在するか。
// has('users', 5) のように第2引数を入れると、リストの件数までチェックできる。
$page->has('propName')
// Propsの中身が期待通りの値か。例: ->where('auth.user.name', '山田太郎')
$page->where('propName', 'value')
// パスワードなど、フロントに渡してはいけないデータが含まれていないことを確認する。
$page->missing('propName')
```

##### バリデーションのアサート
```php
// レスポンスに指定キーのバリデーションエラーがないことを宣言
$response->assertValid();
// レスポンスに指定キーのバリデーションエラーがあることを宣言
$response->assertInvalid(['name', 'email']);
```

#### テストクラスのインスタンス($this)自体から呼び出すアサート
##### 認証のアサート
```php
// ユーザーが認証済みであることを宣言します。
$this->assertAuthenticated($guard = null);
// ユーザーが認証されていないことを宣言します。
$this->assertGuest($guard = null);
// 特定のユーザーが認証済みであることを宣言します。
$this->assertAuthenticatedAs($user, $guard = null);
```


#### ちょっとした解説
メソッド内では、検証したい複数の操作に対して、期待される結果を複数assertで並べられる
```php
$response->assertOk()             // 検問1: そもそも開けたか？
    ->assertInertia(fn ($page) => $page
        ->component('Dashboard')  // 検問2: ページは合ってるか？
        ->has('users', 5)         // 検問3: データ件数は5件か？
        ->where('auth.user.id', 1)// 検問4: ログイン中なのはID:1の人か？
    );
```
これは**「依存関係のあるチェック」**を並べている。  
検問1（アクセス失敗）なら、検問2（ページ名）を調べる必要はない。  
検問2（ページ間違い）なら、検問3（データ件数）を数える必要はない。  
このように、「1つの目的（このページが正しく表示されること）」を達成するための項目を並べるのが正解。  

頻出assert(これだけ知っていれば8割のテストが書ける)
```php
// 1. 【必須】アクセス成功と権限のチェック
// ルーティングの基本。すべてのテストの起点。

// アクセスに成功したか。assertStatus(200)と同じ
$response->assertOk();
// 未ログイン時にログイン画面へ飛ばされたか、保存後に一覧画面へ戻ったか。
$response->assertRedirect('/url');
// 権限がないユーザーがアクセスして、しっかりブロックされたか。assertStatus(403)と同じ
$response->assertForbidden();
// 存在しないIDや、他人のデータにアクセスしたときに404になるか。assertStatus(404)と同じ
$response->assertNotFound();


// 2. 【重要】バリデーションエラーのチェック
// フォーム送信（POST/PATCH）のテストで必須。

// バリデーションに引っかかったか。
// ['name' => 'The name field is required.'] のようにメッセージまで指定も可能。
$response->assertSessionHasErrors(['field']);
// エラーが一切なく、正常に処理されたことの確認。
$response->assertSessionHasNoErrors();

// 4. 【重要】データベースのチェック（TestResponse ではなくテストクラス $this 向け）
// 「保存ボタンを押してリダイレクトはされたけど、実はDBに保存されていなかった」という事故を防ぐ。

// 指定したデータがDBに存在するか。保存処理のテストの最後に必ず書く。
$this->assertDatabaseHas('table', ['column' => 'value']);
// 削除処理のテストで、データが消えたことを確認する。
$this->assertDatabaseMissing('table', ['column' => 'value']);
```
