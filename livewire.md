# Livewire覚書

* 目次
    * [はじめに](#はじめに)
    * [コンポーネント](#コンポーネント)
    * [プロパティ](#プロパティ)
    * [アクション](#アクション)
    * [フォーム](#フォーム)
    * [イベント](#イベント)


<a id="はじめに"></a>
## はじめに

**※コントローラをほぼ完全に回避し、Livewireコンポーネントだけを使用してLaravelアプリケーション全体を構築するアプローチも極めて有力**

[公式サイト](https://livewire.laravel.com/)

### インストール
Laravelアプリのルートディレクトリから、次のコマンドを実行。  
`composer require livewire/livewire`

<a id="コンポーネント"></a>
## コンポーネント

### コンポーネントの生成
コマンド  
`php artisan make:livewire counter-sample`   
※サブディレクトリも生成したい場合  
`php artisan make:livewire posts.create-post`  

このコマンドを実行すると、以下の2つのファイルが生成される。

#### 1.app/Livewire下にコンポーネントのクラス
```php
<?php
namespace App\Livewire;

use Livewire\Component;

class CounterSample extends Component
{
    // render()メソッド内で実行されるview関数で、同時に作成されるコンポーネントのBladeビューが指定されている。
    // コンポーネントクラスと対になるBladeビューを返すだけならば、render()メソッドを完全に省略できる。
    public function render()
    {
        return view('livewire.counter-sample');
    }
}
```

#### 2.resources/views/livewire下にコンポーネントのBladeビュー
```php
<div>
    {{-- Success is as dangerous as failure. --}}
</div>
```

### インラインコンポーネントの生成

コンポーネントがかなり小さい場合は、インラインコンポーネントの使用を推奨。  
インラインコンポーネントは単一のファイルであり、クラスのrender()メソッド内にBladeビュー直接含まれている。  

コンポーネントの生成コマンドに`--inline`フラグを追加する  
例：  
`php artisan make:livewire counter-sample --inline`  

```php
<?php
 
namespace App\Livewire;
 
use Livewire\Component;
 
class CounterSample extends Component
{
    public function render()
    {
        return <<<'HTML' 
        <div>
            {{-- Your Blade template goes here... --}}
        </div>
        HTML;
    }
}

```

### カウンター機能を追加してみる

```php
<?php
 
namespace App\Livewire;
 
use Livewire\Component;
 
class CounterSample extends Component
{
    // Livewireコンポーネントには、データを格納するプロパティがある。
    // プロパティを追加するには、コンポーネントクラスでパブリック変数を宣言。
    // パブリック変数とその値は、ビューコンポーネント内で自動的に利用できるようになる。
    public $count = 1;
 
    // パブリックメソッドはブラウザからトリガーできる。
    public function increment()
    {
        $this->count++;
    }
 
    public function decrement()
    {
        $this->count--;
    }
 
    // レンダリングをするrenderメソッド
    // renderメソッドの主な実行タイミング
    // 1.最初のレンダリング時。プロパティには初期値が適用される。
    // 2.プロパティの値が変更された時。※トリガーされたメソッド内で発生した場合は、そのメソッドの完了まで待機する。
    // ※再レンダリング時はDOMの差分(textContent部分など)のみ更新される。
    public function render()
    {
        return view('livewire.counter-sample');
    }
}
```
```php
// コンポーネントのルートは1つの要素だけでなければならない(複数だとエラー)。
// HTMLのコメントは別の要素としてカウントされるので、ルート要素の中に入れる。
// フルページ・コンポーネントをレンダリングする際、ルート要素の外側にレイアウト・ファイル用の名前付きスロットを置くことができる。これらはコンポーネントがレンダリングされる前に削除される。
<div>
    // パブリック変数へのアクセス。
    <h1>{{ $count }}</h1>
    // クリックイベントでパブリックメソッドをトリガー
    <button wire:click="increment">+</button>
    <button wire:click="decrement">-</button>
</div>
```

#### プロパティ以外のデータを、明示的にビューへ渡すこともできる。  
例：  

```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
 
class CreatePost extends Component
{
    public $title;
 
    public function render()
    {
        return view('livewire.create-post', [
            'author' => Auth::user()->name,
        ]);
    }
}
```
```php
<div>
    <h1>Title: {{ $title }}</h1>
 
    <span>Author: {{ $author }}</span>
</div>
```

### ループ表示
テンプレート内で@foreachを使用する場合、ループによってレンダリングされる要素に一意のwire:key属性を追加する必要がある。  
※wire:key属性が存在しないと、ループが変更されたときにLivewireは古い要素を新しい位置に適切にマッチさせることができず、問題を引き起こす可能性がある。  
```php
<div>
    @foreach ($posts as $post)
        // 例：投稿の配列をループし、wire:key属性を投稿のIDに設定する  
        <div wire:key="{{ $post->id }}"> 
            <!-- ... -->
        </div>
    @endforeach
</div>
```

ループによってLivewireコンポーネントをレンダリングする場合
```php
<div>
    @foreach ($posts as $post)
        // 方法1.コンポーネント属性:keyとしてキーを設定する
        <livewire:post-item :$post :key="$post->id">
        // 方法2.@livewireディレクティブを使用し、第3引数としてキーを渡す
        @livewire(PostItem::class, ['post' => $post], key($post->id))
    @endforeach
</div>
```


### 入力をプロパティにバインドする
Livewire の最も強力な機能の 1 つは「データ バインディング」です。これは、ページ上のフォーム入力とプロパティを自動的に同期させる機能です。

※HTML ディレクティブを学ぶときに詳しくやるかな


### アクションの呼び出し

アクションは、Livewire コンポーネント内のメソッドであり、ユーザー操作を処理したり、特定のタスクを実行したりします。多くの場合、ページ上のボタンのクリックやフォームの送信に応答するのに役立ちます。


※HTML ディレクティブを学ぶときに詳しくやるかな



### コンポーネントのレンダリング
コンポーネントをページ上にレンダリングする方法は2つある。

1. 既存のBladeビューに挿入する
1. フルページコンポーネントとしてルートに直接割り当てる


### コンポーネントを組み込む
```php
<html>
<head>
</head>
<body>
    // コンポーネントをBladeビューに挿入してレンダリングする、コンポーネントタグ。
    <livewire:create-post />
    // サブディレクトリ内にある場合
    <livewire:editor-posts.create-post />

    // コンポーネントの$titleプロパティに初期値を渡す場合
    <livewire:create-post title="Initial Title" />
    // 動的な値や変数を渡す必要がある場合。属性値にPHP式を記述できる。
    <livewire:create-post :title="$initialTitle" />
</body>
</html>
```
```php
<?php
 
namespace App\Livewire;
 
use Livewire\Component;
 
class CreatePost extends Component
{
    public $title;

    // mountメソッドでプロパティに初期値を割り当てる
    // コンストラクタみたいなもの
    // 省略すると、渡された名前と一致するプロパティを自動的に設定する
    public function mount($title = null)
    {
        $this->title = $title;
    }
 
    // ...
}
```


### フルページコンポーネント
まず、フルページコンポーネントに必要なレイアウトを作成する。

resources/views/components/layouts/app.blade.phpの作成コマンド  
`php artisan livewire:layout`  
※デフォルトでは、Livewireはこのファイルを自動的に検索して使用する。  
コンポーネントは{{ $slot }}部分にレンダリングされる。    
Livewire3以降では必要なフロントエンドアセット(@livewireStylesと@livewireScripts)が自動的に挿入される。     

```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
 
        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }}
    </body>
</html>
```
ルートにコンポーネントを直接割り当て、ブラウザでパスにアクセスすると、CreatePostコンポーネントがフルページコンポーネントとしてレンダリングされる。
```php
use App\Livewire\CreatePost;
 
Route::get('/posts/create', CreatePost::class);
```

特定のコンポーネントに別のレイアウトを使用するには、Livewireの#[Layout]属性にカスタムレイアウトの相対ビューパスを渡す。  
<title>{{ $title ?? 'Page Title' }}</title>部分にタイトルを設定するには、#[Title]属性にページタイトルを渡す。  
コンポーネントのプロパティを使用するタイトルなど、動的なタイトルを渡す必要がある場合は、->title()メソッドを使用する。  

```php
<?php

namespace App\Livewire;
 
use Livewire\Attributes\Layout;
use Livewire\Component;

// 方法1
#[Layout('layouts.app')] // resources/views/layouts/app.blade.php
#[Title('Create Post')] 
class CreatePost extends Component
{
    // ...

    // 方法2
    #[Layout('layouts.app')] 
    #[Title('Create Post')] 
    public function render()
    {
        return view('livewire.create-post')
            ->title('Create Post'); 
    }
}
```

####　ルートパラメータへのアクセス
```php
use App\Livewire\ShowPost;
 
Route::get('/posts/{id}', ShowPost::class);
```
```php

<?php
 
namespace App\Livewire;
 
use App\Models\Post;
use Livewire\Component;
 
class ShowPost extends Component
{
    public Post $post;

    // 引数をルートパラメータ名と一致させる
    public function mount($id) 
    {
        $this->post = Post::findOrFail($id);
    }
 
    public function render()
    {
        return view('livewire.show-post');
    }
}
```


<a id="プロパティ"></a>
## プロパティ

* プロパティは、Livewireコンポーネント内のデータを保存および管理する。  
* プロパティはコンポーネントクラスのパブリック変数として定義される。  
* サーバー側とクライアント側の両方でアクセスおよび変更できる。  


```php
<?php
 
namespace App\Livewire;
 use Illuminate\Support\Facades\Auth;
use Livewire\Component;
 
class ManageTodos extends Component
{
    public $todos = [];
 
    public $todo = '';
 
    public function addTodo()
    {
        $this->todos[] = $this->todo;
        // プロパティのリセット※mount()メソッドが呼び出される前の状態にリセットする。
        $this->reset('todo'); 

        // ※上記の処理を一括で行う方法
        $this->todos[] = $this->pull('todo'); 
    }

    public function mount()
    {
        // プロパティの初期化
        // mount()メソッド内でプロパティの初期値を設定できる。
        $this->todos = Auth::user()->todos; 
    }
 
    // ...
}
```

### サポートされているプロパティタイプ
* public $todos = []; // Array
* public $todo = ''; // String
* public $maxTodos = 10; // Integer
* public $showTodos = false; // Boolean
* public $todoFilter; // Null
* BackedEnum
* Illuminate\Support\Collection
* Illuminate\Database\Eloquent\Collection
* Illuminate\Database\Eloquent\Model
* DateTime
* Carbon\Carbon
* Illuminate\Support\Stringable


### プロパティをロックする
```php
use Livewire\Attributes\Locked;
use Livewire\Component;
 
class UpdatePost extends Component
{
    // クライアント側でプロパティが変更されるのを防ぐ。
    // ユーザがフロントエンドで$idを変更しようとするとエラーが出る。
    #[Locked] 
    public $id;
 
    // ...
}
```

### 計算プロパティ
```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
 
class ShowTodos extends Component
{
    // Eloquent制約はリクエスト間で保持されないので、計算プロパティとして定義する。
    // そうすると、その場で評価される動的プロパティ($this->todos)としてアクセスできる
    // クラス内でも、$this->todos()のようにメソッドとして呼ぶよりパフォーマンス上の利点がある。
    #[Computed] 
    public function todos()
    {
        return Auth::user()
            ->todos()
            ->select(['title', 'content'])
            ->get();
    }
 
    public function render()
    {
        return view('livewire.show-todos');
    }
}
```

<a id="アクション"></a>
## アクション


ボタンのクリックやフォームの送信などのフロントエンドのインタラクションによってトリガーできる、コンポーネント上のメソッド。


```php
// 「wire:任意のブラウザイベント名」でイベントリスナーを設定
// 「wire:confirm」で確認アラートを表示
<button
    type="button"
    wire:click="delete"
    wire:confirm="Are you sure you want to delete this post?"
>
    Delete post 
</button>
```

### 特定のキーをリッスンする
```php
// ユーザーが検索ボックスに入力した後にEnterキーを押したときに検索を実行する
<input wire:model="query" wire:keydown.enter="searchPosts">
// キーの組み合わせで設定
<input wire:keydown.shift.enter="...">
```
使用可能なすべてのキー修飾子のリスト
|Modifier|Key|
|-|-|
|.shift|Shift|
|.enter|Enter|
|.space|Space|
|.ctrl|Ctrl|
|.cmd|Cmd|
|.meta|Cmd(Mac), Windows key(Windows)|
|.alt|Alt|
|.up|Up arrow|
|.down|Down arrow|
|.left|Left arrow|
|.right|Right arrow|
|.escape|Escape|
|.tab|Tab|
|.caps-lock|Caps Lock|
|.equal|Equal, =|
|.period|Period, .|
|.slash|Forward Slash, /|



※Livewireは、wire:submitアクションが処理されている間、submitボタンと<form>要素内のすべてのフォーム入力を自動的に無効にする。これにより、フォームが誤って2回送信されることはない。


### パラメータの受け渡し
```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;
 
class ShowPosts extends Component
{
    // ここでパラメータを受取る
    // 注意：アクションパラメータはHTTPリクエスト入力と同じで信頼できない。
    public function delete($id)
    {
        $post = Post::findOrFail($id);
 
        $this->authorize('delete', $post);
 
        $post->delete();
    }
 
    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```
```php
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>
            // ここでパラメータを渡す
            <button wire:click="delete({{ $post->id }})">Delete</button> 
        </div>
    @endforeach
</div>

```


### 依存性注入  
例：
```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Repositories\PostRepository;
 
class ShowPosts extends Component
{
    // ここで依存性注入 
    // パラメータも受取っている。
    public function delete(PostRepository $posts, $postId) 
    {
        $posts->deletePost($postId);
    }
 
    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```
```php
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>
 
            <button wire:click="delete({{ $post->id }})">Delete</button> 
        </div>
    @endforeach
</div>
```


### マジックアクション
メソッドを定義せずにコンポーネント内で一般的なタスクを実行できる。  
テンプレートで定義されたイベントリスナー内で使用。  

```php
// コンポーネントのrender()メソッドを実行する。
// wire:modelによるバインディングなどがサーバーに適用されることに注意
<button type="button" wire:click="$refresh">...</button>

// 子コンポーネントから親コンポーネントのアクションを呼び出す。
<button wire:click="$parent.removePost({{ $post->id }})">Remove</button>

// テンプレートから直接コンポーネントのプロパティを更新する。
// 更新するプロパティと新しい値を引数として指定する。
<button wire:click="$set('query', '')">Reset Search</button>

// コンポーネントのブール型プロパティの真偽値を切り替える。
// 切り替えるプロパティを引数として指定する。
<button wire:click="$toggle('sortAsc')">
    Sort {{ $sortAsc ? 'Descending' : 'Ascending' }}
</button>
```

### プライベートメソッド
コンポーネントクラス内のすべてのパブリックメソッドは、クライアントから呼び出すことができる。  
メソッドをクライアント側から呼び出せないようにするには、privateかprotectedとして定義する。  



<a id="フォーム"></a>
## フォーム

### コンポーネント内のシンプルなフォーム
```php
<?php
 
namespace App\Livewire;
 
use Livewire\Component;
use App\Models\Post;
 
class CreatePost extends Component
{
    #[Validate('required')]
    public $title = '';
 
    #[Validate('required')]
    public $content = '';
 
    public function save()
    {
        $this->validate(); 

        Post::create(
            $this->only(['title', 'content'])
        );
 
        session()->flash('status', 'Post successfully updated.');
 
        return $this->redirect('/posts');
    }
 
    public function render()
    {
        return view('livewire.create-post');
    }
}
```
```php
// submit時に$titleプロパティと$contentプロパティをバインドしする。
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror 
    </div>
 
    <input type="text" wire:model="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror 
    </div>
 
    <button type="submit">Save</button>
</form>
```

### フォームオブジェクト

例えば大規模なフォームを扱っていて、そのすべてのプロパティや検証ロジックなどを別のクラスに抽出したい場合に使う。
フォームオブジェクトを使用してすべてのフォーム関連コードを別のクラスにグループ化し、コンポーネントクラスをよりクリーンな状態に保つ。

作成コマンド  
例：app/Livewire/Forms/PostForm.phpを作成
`php artisan livewire:form PostForm`

※ゆくゆく必要になったら学ぶかも

### updatedメソッドでリアルタイムフォーム保存
※ゆくゆく必要になったら学ぶかも

### 入力フィールドをBladeコンポーネントに抽出する
```php
<form wire:submit="save">
    // 使用例
    <x-input-text name="title" wire:model="title" /> 
    // 使用例
    <x-input-text name="content" wire:model="content" /> 
 
    <button type="submit">Save</button>
</form>
```
```php
<!-- resources/views/components/input-text.blade.php -->
 
// 呼び出し時に渡したname属性値を取り出して$nameとしてこのコンポーネント内で利用できるようにする。
@props(['name'])
 
// 呼び出し時に渡した属性と属性値を$attributesで受取る
// ただし@props()で指定した属性は、$attributesでは受け取らない。
<input type="text" name="{{ $name }}" {{ $attributes }}>
 
<div>
    @error($name) <span class="error">{{ $message }}</span> @enderror
</div>
```


<a id="イベント"></a>
## イベント

開発者は、任意の名前のイベントを定義して発行できる  
コンポーネント内のどこからでもdispatch()メソッドを使用してイベントを発行できる  
ページ上の他のコンポーネントからそのイベントを購読できる。？？？？  

### イベントのディスパッチ
コンポーネントからイベントを発行するには、dispatch()メソッドを呼び出して、イベント名と、イベントと一緒に送信する追加データを渡す。
```php
use Livewire\Component;
 
class CreatePost extends Component
{
    public function save()
    {
        // ...
 
        // dispatch()メソッドでpost-createdイベントを発行
        // このイベントを購読しているページ上の他のすべてのコンポーネントに通知される
        // メソッドの第二引数としてデータを渡すことで、イベントとともに追加データを渡すことができる
        $this->dispatch('post-created', title: $post->title);
    }
}

```






