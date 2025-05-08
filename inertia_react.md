# Inertia+Reactの覚書

* 目次
    * [はじめに](#はじめに)
    * [inertia](#inertia)
    * [React](#React)



<a id="はじめに"></a>
## はじめに


## Laravel 12 React Starter Kitの始め方
1. PHPとComposerをインストール
1. `composer global require laravel/installer`を実行-Composer経由でLaravelインストーラーをインストール
1. `laravel new example-app`を実行-Laravelの新規プロジェクト作成
1. starter kitを聞かれたらReactを選択
1. 認証方法を聞かれたらLaravel's build-in authenticationを選択
1. テストに使うフレームワークを聞かれたらPestを選択
1. `npm install`と`npm run build`を実行するか聞かれたらyesを選択
1. `composer require askdkc/breezejp --dev`を実行-Breezejpをインストール
1. `php artisan breezejp`を実行-必要な言語ファイルの出力

## git clone時
`composer install`を実行-バックエンド（PHP）の依存関係をインストール  
`npm install`を実行-フロントエンド（JavaScript）の依存関係をインストール(package.jsonを元にnode_modulesにインストール)  
`cp .env.example .env`を実行-.env.exampleファイルをコピーして.envファイルを作成
`php artisan key:generate`を実行-.env.exampleをコピーして.envを作成してもアプリケーションキーは設定されていないので、このコマンドで設定する。

### 開発環境で開発を始める場合:
`npm run dev`を実行-開発サーバーを起動し、ソースコードの変更を監視して自動的に再ビルド・ブラウザ更新を行ってくれる。  
※`composer run dev`で、`npm run dev`と`php artisan serve`を同時に実行できる  
### 本番環境にデプロイする場合、または開発環境で本番ビルドを試したい場合:
`npm run build`を実行  




## .prettierrc設定
コードフォーマッターPrettierがどのようにコードをフォーマットするかのルールを指定する
```json
  {
    // インデント幅をスペース2つ分に設定
    "tabWidth": 2,
    //  複数行にわたる配列やオブジェクトの末尾にカンマが自動で挿入される
    "trailingComma": "es5",
    // アロー関数の引数が1つの場合でも、常に括弧 () で囲むように設定
    "arrowParens": "always",
  }
```

`npm run format`-設定を適用する

## .editorconfig設定
インデント幅などを設定できる
```
[*.{json,js,jsx,ts,tsx}]
indent_size = 2
```

## pushする前に
この5つのコマンドで確認  
`npm run lint`  
`npm run format`  
`npm run types`  
`php artisan test`  
`./vendor/bin/pint`  


<a id="inertia"></a>
## inertia

[inertiaドキュメント](https://inertiajs.com/)

### フルページコンポーネント

* 最初のリクエストは、通常のフルページブラウザリクエスト
* 以降のリクエストはすべてJSON 応答
* 通常、アプリケーション内の各ページには専用のコントローラー / ルートと、対応するJavaScriptコンポーネントが存在する。
* フルページコンポーネントを作成がInertiaの基本的なアプローチ
* 各Laravelルートが特定のReactコンポーネントに対応する
* Laravel ルーティング: 通常通り、routes/web.php などでルートを定義
* Laravel コントローラー:リクエストを処理し、必要なデータを準備
* return Inertia::render() を使ってレスポンスを返す。このメソッドの第一引数には、表示したいReactコンポーネントの名前を指定し、第二引数にはそのコンポーネントに渡したいデータ（props）を連想配列で指定。
* フルページコンポーネント内でUIを部分的なコンポーネントに分割し、入れ子にすることは、コードを整理し、保守性や再利用性を高めるための標準的で推奨されるプラクティス。
* Inertia + React 環境では、Reactコンポーネントは手動で作成するのが標準的な方法

構成
* resources/js/Pages:
* Laravelのコントローラーから Inertia::render() で指定されるフルページコンポーネント（.tsx ファイル）を格納
* 例: Users/Index.tsx, Users/Show.tsx, Dashboard.tsx, Auth/Login.tsx
* resources/js/Components:
* 再利用可能な部分的なUIコンポーネント（入れ子になるコンポーネント、.tsx ファイル）を格納
* 例: Button.tsx, Modal.tsx, TextInput.tsx, UserAvatar.tsx
* resources/js/Layouts (もし使う場合):
* アプリケーション全体で共有される永続レイアウトコンポーネント（.tsx ファイル）を格納
* 例: AuthenticatedLayout.tsx, GuestLayout.tsx


#### 例1
コントローラー
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Inertia\Inertia; // Import Inertia

class UserController extends Controller
{
    public function index()
    {
        $users = User::all();

        // 'Users/Index' という名前のReactコンポーネントを
        // 'users' というプロップスと共にレンダリングする
        return Inertia::render('Users/Index', [
            'users' => $users,
        ]);
    }
}
```
コンポーネント
```jsx
// resources/js/Pages/Users/Index.jsx
import React from 'react';
import Layout from '@/Layouts/Layout'; // 例: 共通レイアウト

export default function Index({ users }) { // コントローラーから渡された 'users'propsを受け取る
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name} ({user.email})</li>
        ))}
      </ul>
    </div>
  );
}

// オプション: 永続レイアウトを設定
Index.layout = page => <Layout children={page} title="User List" />;
```

#### 例2
コントローラー
```php
use Inertia\Inertia;

class UserController extends Controller
{
    public function show(User $user)
    {
        // ページをレンダリング
        // User/Show ページは通常、resources/js/Pages/User/Show.jsxにあるファイルに対応する
        return Inertia::render('User/Show', [
          'user' => $user
        ]);
    }
}
```
コンポーネント
```jsx
// resources/js/Pages/User/Show.jsx

import Layout from './Layout'
import { Head } from '@inertiajs/react'

// ページはアプリケーションのコントローラーからpropsとしてデータを受け取る。
export default function Welcome({ user }) {
  return (
    // ページコンテンツを<Layout>コンポーネントで囲んでいる
    <Layout>
      <Head title="Welcome" />
      <h1>Welcome</h1>
      <p>Hello {user.name}, welcome to your first Inertia app!</p>
    </Layout>
  )
}
```
レイアウトの作成
```jsx
import { Link } from '@inertiajs/react'

// <Layout>コンポーネント
export default function Layout({ children }) {
  return (
    <main>
      <header>
        <Link href="/">Home</Link>
        <Link href="/about">About</Link>
        <Link href="/contact">Contact</Link>
      </header>
      <article>{children}</article>
    </main>
  )
}
```

### 永続的なレイアウト
ポッドキャストのウェブサイトでオーディオプレーヤーを再生し続けたい場合や、ページを移動してもサイドバーナビゲーションのスクロール位置を維持したい場合など


### リダイレクト
```php
class UsersController extends Controller
{
    public function index()
    {
        return Inertia::render('Users/Index', [
            'users' => User::all(),
        ]);
    }

    public function store(Request $request)
    {
        User::create(
            $request->validate([
                'name' => ['required', 'max:50'],
                'email' => ['required', 'max:50', 'email'],
            ])
        );

        // リダイレクトを返す
        return to_route('users.index');
    }
}
```

### 速記ルート
「FAQ」や「About」ページなど、対応するコントローラー メソッドを必要としないページがある場合は、Route::inertia()メソッドを介してコンポーネントに直接ルーティングできる。
```jsx
Route::inertia('/about', 'About');
```


### タイトルとメタ
ヘッドコンポーネント
<head>要素を追加するには、<Head>コンポーネントを使用
```jsx
import { Head } from '@inertiajs/react'

<Head>
  <title>Your page title</title>
  <meta name="description" content="Your page description" />
</Head>

// タイトルだけなら略して書ける
<Head title="Your page title" />
```

### リンク
* Inertiaアプリ内で他のページへのリンクを作成するには、通常、Inertia<Link> コンポーネントを使う
* アンカーリンクのラップで、ページ全体の再読み込みを防ぐ。
* デフォルトでは、Inertia はリンクをアンカー要素としてレンダリングするが、
* asプロパティを使用してタグを変更することもできる
* dataプロパティを使用してリクエストに追加データを指定できる
```jsx
import { Link } from '@inertiajs/react'

<Link href="/logout" method="post" as="button" data={{ foo: bar }}>Logout</Link>
```

### 手動訪問
```ts
import { router } from '@inertiajs/react'

// 全てurl引数のみ必須

// 汎用メソッド
router.visit(url, {
  method: 'get',
  data: {},
  replace: false,
  preserveState: false,
  preserveScroll: false,
  only: [],
  except: [],
  headers: {},
  errorBag: null,
  forceFormData: false,
  queryStringArrayFormat: 'brackets',
  async: false,
  showProgress: true,
  fresh: false,
  reset: [],
  preserveUrl: false,
  prefetch: false,
  onCancelToken: cancelToken => {},
  onCancel: () => {},
  onBefore: visit => {},
  onStart: visit => {},
  onProgress: progress => {},
  onSuccess: page => {},
  onError: errors => {},
  onFinish: visit => {},
  onPrefetching: () => {},
  onPrefetched: () => {},
})

// 特化メソッド
router.get(url, data, options?: Omit<VisitOptions, 'method' | 'data'>)
router.post(url, data, options?: Omit<VisitOptions, 'method' | 'data'>)
router.put(url, data, options?: Omit<VisitOptions, 'method' | 'data'>)
router.patch(url, data, options?: Omit<VisitOptions, 'method' | 'data'>)
router.delete(url, options?: Omit<VisitOptions, 'method'>)
// Uses the current URL
router.reload(options?: Omit<VisitOptions, 'preserveScroll' | 'preserveState'>)
```


### フォームの送信  

```jsx
import { useForm } from '@inertiajs/react'

// useForm は、フォームのデータを内部で状態として管理する
const { data, setData, post, processing, errors } = useForm({
  // 各データ名と、初期値を指定
  email: '',
  password: '',
  remember: false,
})

// フォーム送信をインターセプトしてからInertiaを使ってリクエストを行う
function submit(e) {
  // フォームのデフォルト動作をキャンセルする
  e.preventDefault()
  post('/login')
}

return (
  <form onSubmit={submit}>
    <input type="text" value={data.email} onChange={e => setData('email', e.target.value)} />
    {errors.email && <div>{errors.email}</div>}
    <input type="password" value={data.password} onChange={e => setData('password', e.target.value)} />
    {errors.password && <div>{errors.password}</div>}
    <input type="checkbox" checked={data.remember} onChange={e => setData('remember', e.target.checked)} /> Remember Me
    <button type="submit" disabled={processing}>Login</button>
  </form>
)
```

### 部分的なリロード
* コントローラーは通常通り実行される
* サーバーから特定のpropsのみを受け取って更新
* Reactが差分のみを再レンダリング


```jsx
import { router } from '@inertiajs/react'

// props のキーに対応するキーの配列を指定
// 特定のpropsのみ
router.reload({ only: ['users'] })
// 特定のpropsを除く
router.reload({ except: ['users'] })
```

```php
// Partial reload の際、クロージャで囲むと必要なデータだけを遅延評価して取得できる。
return Inertia::render('Users/Index', [
    'users' => fn () => User::all(),
    'companies' => fn () => Company::all(),
]);

return Inertia::render('Users/Index', [
    // 常に評価される
    'users' => User::all()、
    // 必要なときだけ評価される
    'users' => fn () => User::all()、

    // only で呼ばれた時だけのピンポイント登板。
    'users' => Inertia::optional(fn () => User::all())、
    // 基本スタメンだけど、except でお休みできるし、only で呼ばれなければ出番なし。
    'users' => Inertia::always(User::all())、
]);
```


### 大事っぽい

usePage
Link
useForm(内部でrouter使用)
router
useRemember

### 画像
結論・推奨  
ロゴ、アイコン、UI要素の一部として使う静的な画像:  
resources/js/assets/images/ に置き、Reactコンポーネント内で import するのがベストプラクティスです（特にVite使用時）。  
ユーザーがアップロードした画像、DBなどで管理される動的な画像:  
storage/app/public/ に保存し、Laravel側で生成した公開URLを props でReactに渡し、そのURLを src に指定します。  
非常に単純なケースや、ビルドプロセスを通したくない場合:  
public/images/ に置き、ルートからの絶対パス /images/your-image.png で指定することも可能です。  
多くの場合、UIコンポーネントで直接使う静的画像は1番目の方法（resources/js 以下に置いて import）を使うのが最もモダンで堅牢な方法と言えるでしょう。  

<a id="React"></a>
## React

[クイックスタート](https://ja.react.dev/learn)


* コア概念
  * Reactアプリはコンポーネント(UIの部品)で構成されている。
  * コンポーネントは、JSXを返すJavaScript関数として定義する。
  * JSXは、JavaScriptがHTMLに化けたマークアップ構文。JSの値(式)を`{}`で埋め込める。
  * 埋め込んだ値がtrue, false, true, null, undefinedの場合は何もレンダリングしない。
  * JSXの配列は兄弟要素としてレンダリングされる
  * 配列内の各要素をmap等でレンダリングする際には、兄弟の中でそれを一意に識別するためのkey属性(文字列または数値)を設定する必要がある。
  * 単一の要素しか返せないので、透明なラッパーとして<></>のようなタグで囲むこともある
  * export default キーワードは、ファイル内のメインコンポーネントを指定している。


### イベント処理
コンポーネントの中でイベントハンドラ関数を宣言することで、イベントに応答できる
```jsx
const MyButton = () => {

  const handleClick = () => {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

### state
コンポーネントに情報を「記憶」させる
```jsx
// React から useState をインポート
import { useState } from 'react';

// 同じコンポーネントを複数の場所でレンダーした場合、それぞれが独自のstateを持つ。
const MyButton = () => {
  // useStateからはstateと、それを更新するための関数を得られる。
  // 名前は何でもいいが、慣習的には [something, setSomething] のように記述する。
  // useStateには初期値を渡す
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // stateが更新されると、再レンダリングがなされる(ただし関数の完了までは待つ)
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

### フック

useで始まる関数は、フック (Hook) と呼ばれる。
* useState
* useEffect
* useContext
* useRef
* useCallback
* useMemo


### props
コンポーネントはpropsと呼ばれる任意の入力を受け取ることができる。

```jsx
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}


export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

