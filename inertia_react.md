# Inertia+Reactの覚書

* 目次
    * [はじめに](#はじめに)
    * [inertia](#inertia)
    * [React](#React)
    * [TSの型](#TSの型)
    * [shadcn](#shadcn)




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

### 基本

* Laravel(厨房)
* Inertia(ウェイター)
* React(ホール)

基本的に、データ送信側ではInertiaの機能を使い、データ受信側ではそれぞれのフレームワーク(LaravelとReact)の標準的な方法で受け取る

* PHP側
  * Inertia::render() ※基本レンダリング
  * to_route() ※リダイレクト
  * Route::inertia() ※コンポーネントに直接ルーティング
  * Inertia::defer() ※特定のページデータの読み込みを最初のページレンダリング後まで延期
  * Inertia::merge() ※propsを上書きする代わりに、マージする。ページネーションや無限スクロールで使う。
  * Inertia::deepMerge() ※Inertia::merge()の深い処理版
  * Inertia::share() ※共有データ。認証ユーザー情報、フラッシュメッセージ等、全ページで利用可能なデータを渡す
  * HandleInertiaRequests ※ミドルウェア。Inertia::share() の主要な実装場所。
* TypeScript側
  * Home.layout = page => \<Layout ... /> ※永続レイアウト
  * \<Head /> ※コンポーネント
  * \<Link>/</Link> ※コンポーネント
  * router ※手動訪問
  * router.reload({ only: ['users'] }) ※Partial reloads
  * useForm() ※フォーム。入力状態をブラウザ履歴に保存。errorsプロパティを使ったバリデーションエラーの表示。
  * usePoll(2000) ※ミリ秒単位のポーリング
  * Linkコンポーネントのprefetch ※先読みでデータを取得し30秒間キャッシュ
  * WhenVisible ※無限スクロール時のデータの遅延読み込み
  * useRemember ※UIの状態をブラウザ履歴に保存(フォームデータの場合はuseFormを使ったほうが簡潔)
  * usePage ※表示中ページのURLやページコンポーネントの情報を取得できる。 



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
#### useFormフック
```tsx
import { useForm } from '@inertiajs/react'

const MyForm = () => {

  type LoginForm = {
    email: string,
    password: string,
    remember: boolean,
  }
  
  // useFormは、フォームのデータを内部で状態として管理する(内部ではrouterを使用してHTTPリクエストを送信している)
  // 任意で第一引数に一意のフォームキーを渡し、フォームデータとエラーを自動的にブラウザ履歴に保存できる
  const { data, setData, post, processing, errors } = useForm<LoginForm>('yourFormKey', {
    // 各データ名と、初期値を指定
    email: '',
    password: '',
    remember: false,
  })

  // フォーム送信をインターセプトしてからInertiaを使ってリクエストを行う
  const submit = (e) => {
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
}

export default MyForm;
```


#### useFormフックの戻り値(プロパティとメソッド)の詳細
```tsx
type LoginForm = {
  email: string,
  password: string,
  remember: boolean,
}

const {
  // プロパティ

  // フォームの現在の入力値を保持するオブジェクト。
  // useFormを初期化した際のデフォルト値、またはsetDataによって更新された値が格納される。
  // TFormはフォームで扱うデータの型(ここではLoginForm)を指す。
  // dataは内部stateを参照したもの。そしてsetDataはその内部stateを更新する。よってsetDataによるdata更新で再レンダリングがなされる
  // 再レンダリング時にdata値とinput入力値の整合性を保つため、input要素のvalue属性にdata値(data.emailのようにアクセス)を設定する。
  data: TForm,
  // フォームのデータが初期状態（デフォルト値、または reset()やsetDefaults()で最後に設定された値）から変更されたかどうかを示すブール値
  // フォームが変更された場合にのみ「保存」ボタンを有効化する、などのUI制御に使える。
  isDirty: boolean,
  // サーバーサイド（Laravel）でのバリデーションエラーメッセージを格納するオブジェクト
  // キーはフォームのフィールド名、値はそのフィールドのエラーメッセージ文字列
  // Partialなので、エラーのないフィールドのキーは存在しない。
  // 例えばerrors.emailでメールアドレスフィールドのエラーメッセージを取得できる。
  // 例：{errors.name && <div className="text-red-500 text-sm mt-1">{errors.name}</div>}
  errors: Partial<Record<keyof TForm, string>>,
  // errorsオブジェクトに何かしらのエラーメッセージが含まれているかどうかを示すブール値。
  // エラーがある場合にエラーメッセージのサマリーを表示したり、UIのスタイルを変更したりするのに使える。
  hasErrors: boolean,
  // フォームが現在サーバーに送信処理中であるかどうかを示すブール値
  // 送信開始時にtrueになり、レスポンスを受け取るとfalseになる。
  // 送信中はボタンを無効化したり、ローディングインジケーターを表示したりするのに使える。
  processing: boolean,
  // ファイルアップロードを伴うフォーム送信の場合に、アップロードの進捗状況を表すオブジェクト。
  // 通常、{ percentage: number, loaded: number, total: number } のようなプロパティを持つ。
  // ファイルアップロードがない場合はnull。
  // ファイルアップロードのプログレスバーを表示するのに使う。
  progress: Progress | null,
  // フォーム送信が成功したかどうかを示すブール値。
  // 送信が成功するとtrueになる。この状態は、フォームがリセットされるか、再度送信が試みられるまで維持される。
  // 送信成功後に永続的なフィードバック（例: 「設定が保存されました」というメッセージの表示）を行うのに使える。
  wasSuccessful: boolean,
  // フォーム送信が直近で成功した場合にtrueになるブール値。
  // wasSuccessfulと似ているが、こちらは一定時間（デフォルトでは2秒）が経過すると自動的にfalseに戻る。
  // 送信成功後に一時的なフィードバック（例: ボタンの見た目を一時的に変える、トースト通知を短時間表示する）を行うのに便利。
  recentlySuccessful: boolean,


  // メソッド

  // フォームのデータを更新するためのメソッド。複数の方法で呼び出せる。詳細は後述。
  setData: setDataByObject<TForm> & setDataByMethod<TForm> & setDataByKeyValuePair<TForm>,
  // フォームデータをサーバーに送信する直前に、そのデータを加工・変換するためのコールバック関数を登録する。
  // 送信前に不要なプロパティを削除したり、特定の形式（例: 日付フォーマット）に変換したり、新しいプロパティを追加したりする場合に使う。
  // 例: transform((data) => ({ ...data, password_confirmation: data.password })) ※パスワード確認フィールドを追加
  transform: (callback: (data: TForm) => object) => void,
  // 現在のdataの値を、フォームの新しいデフォルト値として設定する。reset()を呼び出した際に、この値に戻るようになる。
  setDefaults(): void,
  // 特定のフィールドのデフォルト値を新しい値で設定する。
  setDefaults(field: keyof TForm, value: FormDataConvertible): void,
  // 複数のフィールドのデフォルト値をオブジェクトでまとめて設定する。
  // フォームがマウントされた後、APIから取得したデータなどで初期値を設定し、それを「リセット先」のデフォルト値としたい場合などに使う。
  setDefaults(fields: Partial<TForm>): void,
  // フォームのデータをデフォルト値（useFormの初期値、またはsetDefaultsで設定された値）に戻す。
  // 引数にはリセットしたいフィールドのフィールド名を指定する。例: reset('email', 'password')
  // 引数なしで実行すると全てのフィールドをリセットする。
  reset: (...fields: (keyof TForm)[]) => void,
  // フォームのバリデーションエラーメッセージをクリアする。
  // 引数にはエラーメッセージをクリアしたいフィールドのフィールド名を指定する。例: reset('email', 'password')
  // 引数なしで実行すると全てのフィールドのエラーメッセージをクリアする。
  // ユーザーが入力値を修正した際に、対応するエラーメッセージを手動で消したい場合などに使う。
  clearErrors: (...fields: (keyof TForm)[]) => void,
  // 特定のフィールドに手動でエラーメッセージを設定する。
  setError(field: keyof TForm, value: string): void,
  // 複数のフィールドにエラーメッセージをオブジェクト形式でまとめて設定する。
  // クライアントサイドでの独自のバリデーション結果を表示したい場合や、サーバーからのエラーとは別にエラーを設定したい場合に使う。
  setError(errors: Record<keyof TForm, string>): void,
  // 現在進行中のフォーム送信リクエストをキャンセルする。
  // ユーザーが送信を中断したい場合や、タイムアウト処理などで使える。
  cancel: () => void,
  // フォームデータを指定されたHTTPメソッド（'get', 'post', 'put', 'patch', 'delete'）とURLでサーバーに送信する。
  // options: 送信時の追加オプションを指定できる (後述)。
  // 例: submit('post', '/users', { onSuccess: () => console.log('User created!') })
  // ※内部的にはInertiaのルーター機能を使っている。router.get(...)、router.post(...)といった形で。
  submit: (method: Method, url: string, options?: FormOptions) => void,
  // submit('get', url, options) のショートカット
  get: (url: string, options?: FormOptions) => void,
  // submit('post', url, options) のショートカット
  post: (url: string, options?: FormOptions) => void,
  // submit('put', url, options) のショートカット
  put: (url: string, options?: FormOptions) => void,
  // submit('patch', url, options) のショートカット
  patch: (url: string, options?: FormOptions) => void,
  // submit('delete', url, options) のショートカット
  // ????通常、データ本体はURLパラメータやリクエストボディではなく、URL自体でリソースを指す
  delete: (url: string, options?: FormOptions) => void
} = useForm<LoginForm>('yourFormKey', {
  email: '',
  password: '',
  remember: false,
});


// setDataの呼び出し方
// 1.特定のフィールドの値を更新する。
setData('fieldName', value)
// 例: 
setData('email', e.target.value)
// 2.複数のフィールドをオブジェクトでまとめて更新する。
setData({ fieldName1: value1, fieldName2: value2 })
// 例: 
setData({ name: 'Taro', age: 30 })
// 3.現在のデータに基づいて新しいデータを返すコールバック関数を使って更新する。
setData(previousData => ({ ...previousData, fieldName: newValue }))
// 例: 
// ※jsのオブジェクトの値は後勝ち
setData(prev => ({ ...prev, count: prev.count + 1 }))


// options:で送信時の追加オプションを指定できる 
// optionsはオブジェクトで、フォーム送信の挙動を細かく制御するためのコールバック関数や設定が含まれる。
// 代表的なもの：
post(url, {
  // リクエストが成功した場合に実行されるコールバック。
  onSuccess: (page: Page) => void,
  // リクエストが失敗し、バリデーションエラーなどが返された場合に実行されるコールバック。
  // フォーム送信エラー時に特別な処理（通知表示、ロギング、特定UIの制御など）を行いたい場合に使用します。
  // 引数でエラーオブジェクトを受け取れる。
  onError: (errors: Record<string, string>) => void,
  // リクエストが成功したか失敗したかに関わらず、完了時に実行されるコールバック。
  onFinish: (visit: Visit) => void,
  // リクエスト開始時に実行されるコールバック。
  onStart: (visit: Visit) => void,
  // trueの場合、フォーム送信後もコンポーネントのローカルステートを保持する。
  // デフォルトは false（POST/PUT/PATCH/DELETEの場合）または true（GETの場合）。
  preserveState: boolean,
  // trueの場合、フォーム送信後にページのスクロール位置を保持する。
  preserveScroll: boolean,
  // trueの場合、フォーム送信成功時にフォームデータをリセットする。デフォルトは true。
  resetOnSuccess: boolean,
})
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

### usePage
Inertia.jsが提供するReactのカスタムフック。  
Laravel（サーバーサイド）から渡されたデータにReactコンポーネント内からアクセスするための橋渡し役。  
LaravelのコントローラからInertiaに渡されるデータは、「ページオブジェクト」という特定の構造を持っている。  
usePageフックを使うと、どのコンポーネントからでもこのページオブジェクトにアクセスできる。  

```tsx
import { type SharedData } from '@/types';
import { Head, Link, usePage } from '@inertiajs/react';

export default function Welcome() {
  // <SharedData>はTypeScriptのジェネリクス機能。usePageフックが返すpropsオブジェクトの型定義を指定している。
  // usePage() を呼び出すと、以下のようなプロパティを持つオブジェクトが返ってくる。
  const page = usePage<SharedData>();

  // 最も重要。Laravelから渡されたデータがすべてここに入っている。
  // コントローラから渡されたデータと、全ページで共有されるデータ（Shared Data）が含まれる。
  console.log(page.props);
  //  現在のページのURL
  console.log(page.url);
  // 現在レンダリングされているReactコンポーネントの名前（例: 'Users/Index')。
  console.log(page.component);
  // アセットのバージョン。アセットが更新されたことを検知するために使う。
  console.log(page.version);

  // JavaScriptの分割代入
  const { auth } = usePage<SharedData>().props;
  return (
    <nav>
      // page.props.auth.userはログイン中のユーザーインスタンス。
      // LaravelのAuth::user()の情報を単なるデータ(JavaScriptオブジェクト)としてフロントエンドに渡したもの。
      // ※通常、LaravelのHandleInertiaRequestsミドルウェア内でその処理が行われる。shareメソッドの中の$request->user()はAuth::user()と同じ。
      {page.props.auth.user ? () : ()}
    </nav>
  );
}

```

情報2  
表示しているページのURLとページコンポーネントの情報を取得できる。  
この情報を利用することで現在表示しているページのリンクにclassを適用したりできる  
サーバーから渡されたプロパティ (props)、とりわけ共有データ (props.shared 内の認証ユーザー情報やフラッシュメッセージなど) にアクセスする」 という点が最も頻繁に使われる重要な役割
```jsx

import { Link, usePage } from "@inertiajs/react";

const NavBar = () => {
    const { url, component } = usePage();
    return (
      // 略
      <Link
          href={route("user")}
          className={url === "/user" ? "active" : ""}
      >
          User
      </Link>
      // 略
    );
};

export default NavBar;
```
### useRemember
* UIの状態をブラウザ履歴に保存する
  * タブコンポーネントで最後に選択されていたタブ。
  * アコーディオンメニューが開いているか閉じているかの状態。
  * リストの表示件数やソート順、フィルター条件など。

※フォームデータの場合はuseFormを使ったほうが簡潔


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
  * JSXは、JavaScriptがHTMLに化けたマークアップ構文。JSの値(関数)を`{}`で埋め込める。
  * 埋め込んだ値がtrue, false, true, null, undefinedの場合は何もレンダリングしない。
  * JSXの配列は兄弟要素としてレンダリングされる
  * 配列内の各要素をmap等でレンダリングする際には、兄弟の中でそれを一意に識別するためのkey属性(文字列または数値)を設定する必要がある。
  * export default キーワードは、ファイル内のメインコンポーネントを指定している。

* コンポーネントの基本
  1. フック: 特定の特殊能力を付与
  1. State/Ref: 内部に値を持たせる
  1. 処理: 内部に処理を持たせる
  1. イベント: 処理発火のきっかけ
  1. Props: 外部からオブジェクトとして値を渡せる
  1. コンポジション: レゴブロックのようにコンポーネントの組み合わせる


### フラグメント
* jsxは単一の要素しか返せないので、透明なラッパーとしてフラグメントで囲むこともある。
* フラグメントにkey属性を指定する必要がある場合は、<React.Fragment key="...">...</React.Fragment>のように明示的に書く必要がある。※リストの要素としてフラグメントをレンダリングする場合など、key属性が必要になることがある。
* key属性が不要な場合は、簡潔な<></>構文を使用できる。



### state
コンポーネントに情報を「記憶」させる  
値が変更されたときに再レンダリングを引き起こしたい場合に使う  
```jsx
// React から useState をインポート
import { useState } from 'react';

// 同じコンポーネントを複数の場所でレンダーした場合、それぞれが独自のstateを持つ。
const MyButton = () => {
  // useStateからはstateと、それを更新するための関数を得られる。
  // 名前は何でもいいが、慣習的には [something, setSomething] のように記述する。
  // useStateには初期値を渡す
  // useStateの初期値は、コンポーネントが最初にレンダリングされるときに一度だけ評価される
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

#### 基本
* useで始まる関数は、フック (Hook) と呼ばれる。※inertiaのuse○○も同じ。
* 関数コンポーネントのトップレベルで呼び出し、コンポーネントに特定の能力やデータをもたらす
* Reactのフック：汎用的なコア機能。部品。
* Inertiaのフック：高レベルな便利機能。内部でReactのフックを使ってる。

#### useState
#### useContext


#### useEffect
```tsx
// 第二引数に配列で渡したstateのうちのどれかが変化したとき、それに伴ったレンダリングの後に第一引数のコールバック関数が実行される。
useEffect(() => {
  setSelectedImageIndex(0); // 最初の画像を選択状態にする
}, [foo, bar]);
```


#### useRef
```tsx
function App() {
  // 値が変更されても再レンダリングを引き起こしたくない時に使う
  // currentプロパティのみを持つオブジェクトを返す。
  // 型と初期値を指定
  const dragItemOriginalIndexRef = useRef<number | null>(null);

  // currentプロパティは書き換えが可能
  // currentプロパティが更新されても再レンダリングは発生しない
  dragItemOriginalIndexRef.current = index;

  const draggingItemOriginalIdx = dragItemOriginalIndexRef.current; 
}
```

#### useCallback
#### useMemo



### props
コンポーネントはpropsと呼ばれる任意の入力を受け取ることができる。  
Reactの関数コンポーネントに渡されるpropsは、常にオブジェクトである。  
たとえpropsを何も渡さない場合でも、関数コンポーネントには空のオブジェクト {} が propsとして渡されている。

```jsx
const MyButton = ({ count, onClick }) => {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}

const MyApp = () => {
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

export default  MyApp;
```

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

* onから始まるイベントハンドラ属性（onClick, onChange, onSubmit など）に指定した関数には、必ずイベントオブジェクトが第一引数として渡される。
* イベントオブジェクトを指す変数名には慣習的にeが使われる(eventやevtのこともある)。
* このイベントオブジェクトには、イベントが発生した要素や、その時の状況に関する様々な情報が含まれている。
* 渡されるイベントオブジェクトの型（種類）が異なる場合がある。
  * onChange (input要素) -> ChangeEvent
  * onClick (多くの要素) -> MouseEvent
  * onSubmit (form要素) -> FormEvent (または SubmitEvent？違うかも)
  * onKeyDown (多くの要素) -> KeyboardEvent
  * など。
* それぞれのイベントオブジェクトは、そのイベント特有のプロパティを持っている。
* 例えば、KeyboardEvent には key や keyCode といったプロパティがあり、MouseEvent には clientX や clientY といったプロパティがある。


例：  
onChange: input要素の値が変更されたときに発生するイベント  
e.target: イベントが発生したDOM要素（この場合は \<input> 要素）そのものを指す  



#### イベントの伝搬を止めるためによく使われるメソッド
* e.preventDefault()（フォームのデフォルト動作をキャンセル）
* e.stopPropagation()（イベントのバブリングを停止）



#### ドラッグ系イベント
これらのハンドラは
```tsx
onDragOver={(e) => handleDragOver(e)}
```
のように使用したとき必ずeにReact.DragEventが渡る

※React.DragEventのevent.currentTargetは、現在実行中のイベントハンドラが属しているDOM要素を常に参照する。


* 自身がドラッグされたときに発火するもの（ドラッグされる要素側で発生）。これらのイベントは、draggable="true" 属性が設定された要素で発生する。
  * onDragStart:
    * タイミング: ユーザーが要素のドラッグを開始したとき。
    * 主な用途:
      * ドラッグするデータの種類と値を event.dataTransfer.setData() で設定する。
      * ドラッグ操作の視覚的なフィードバック（例: ドラッグゴーストのカスタマイズ）の準備。
      * 許可されるドロップ効果 (copy, move, link など) を event.dataTransfer.effectAllowed で設定する。
      * ドラッグ開始時の状態管理（例: isDragging = true）。
  * onDrag:
    * タイミング: 要素がドラッグされている間、継続的に発生します（マウスが少しでも動くたび）。
    * 主な用途:
      * ドラッグ中の位置情報に基づく処理（頻繁に発生するため、重い処理は避けるべき）。
      * 特定の条件下でのドラッグ操作のカスタム制御（あまり一般的ではない）。
  * onDragEnd:
    * タイミング: ドラッグ操作が終了したとき（ドロップが成功したか、キャンセルされたかに関わらず）。
    * 主な用途:
      * ドラッグ操作後のクリーンアップ処理。
      * ドラッグ終了時の状態管理（例: isDragging = false）。
      * ドロップが成功したかどうかを event.dataTransfer.dropEffect で確認し、それに応じた処理を行う（例: 'move' なら元の要素を削除）。
* ドラッグ要素に覆われた（上に乗っかられた）ときに発火するもの（ドロップターゲット要素側で発生）。これらのイベントは、ドロップを受け入れる可能性のある要素（ドロップターゲット）で発生する。
  * onDragEnter:
    * タイミング: ドラッグされた要素が、ドロップターゲットの境界内に初めて入ったとき。
    * 主な用途:
      * ドロップターゲットがアクティブになったことを視覚的に示す（例: 背景色を変える、ボーダーを強調する）。
      * event.preventDefault() を呼び出してドロップを許可する（場合によっては onDragOver だけで十分）。
      * ドラッグされているデータの種類 (event.dataTransfer.types) を確認し、受け入れ可能か判断する。
  * onDragOver:
    * タイミング: ドラッグされた要素が、ドロップターゲットの境界内に乗っている間、継続的に発生します（マウスが少しでも動くたび）。
    * 主な用途:
      * event.preventDefault() を呼び出して、その場所へのドロップを許可する（これが最も重要な役割）。
      * ドラッグ中の視覚的なフィードバックを継続的に更新する（例: マウスカーソルの形状を変える指示）。
      * ドラッグされている要素の位置に応じて、ドロップ先の候補をハイライトするなどの動的な処理。
  * onDragLeave:
    * タイミング: ドラッグされた要素が、ドロップターゲットの境界内から離れたとき。
    * 主な用途:
      * onDragEnter で適用した視覚的なフィードバックを元に戻す（例: 背景色を戻す）。
      * ドロップターゲットが非アクティブになったことを示す。
  * onDrop:
    * タイミング: ドラッグされた要素が、ドロップターゲット上でドロップ（マウスボタンが離された）されたとき。
    * 主な用途:
      * event.preventDefault() を呼び出して、ブラウザのデフォルトのドロップ処理（例: 画像やリンクならファイルを開く/ページ遷移する）を防ぐ。
      * event.dataTransfer.getData() を使って、ドラッグされた要素のデータを取得する。
      * 取得したデータに基づいて、アプリケーションの状態を更新する（例: リストアイテムの並び替え、ファイルのアップロード処理の開始）。
      * ドロップ成功後のクリーンアップや視覚的フィードバック。

##### まとめ表
|イベントハンドラ|発火する側|タイミング|主な役割|
|-|-|-|-|
|onDragStart|ドラッグされる要素|ドラッグ開始時|データ設定、ドラッグ開始処理|
|onDrag|ドラッグされる要素|ドラッグ中、継続的|ドラッグ中の処理（軽量なもの）|
|onDragEnd|ドラッグされる要素|ドラッグ終了時|クリーンアップ、ドラッグ終了処理|
|onDragEnter|ドロップターゲット要素|ドラッグ要素が境界内に初めて入ったとき|視覚的フィードバック開始、受け入れ可否判断|
|onDragOver|ドロップターゲット要素|ドラッグ要素が境界内に乗っている間、継続的|ドロップ許可 (preventDefault)、継続的フィードバック|
|onDragLeave|ドロップターゲット要素|ドラッグ要素が境界内から離れたとき|視覚的フィードバック終了|
|onDrop|ドロップターゲット要素|ドラッグ要素がドロップされたとき|デフォルト処理防止 (preventDefault)、データ取得、状態更新|


### useCallback

一言で言うと「関数の再生成を防ぎ、パフォーマンスを最適化するためのReactフック」





Reactのコンポーネントは、stateやpropsが変更されると再レンダリング（再描画）される。
このとき、コンポーネント内の関数も、デフォルトでは毎回新しく作り直されてしまう。



特定の状況ではパフォーマンスの低下につながる
特に、子コンポーネントに関数をpropsとして渡す場合


子コンポーネントがReact.memoで最適化されていても（propsが変更されない限り再レンダリングしないようにする仕組み）、親から渡される関数が毎回新しいものだと、子コンポーネントは「propsが変わった！」と認識してしまい、不要な再レンダリングが発生してしまいます。


useCallbackの場合、指定した**「依存配列」**の値が変わらない限り、最初に作成した関数インスタンスを再利用します。  
依存配列が空だと、依存配列の値が変わりようがないので、最初に作成した関数インスタンスが再利用され続ける
```tsx
  // もし関数内で外部の変数（stateやprops）を使いたい場合
  const doSomethingWithCount = useCallback(() => {
    console.log("現在のカウント:", count);
  }, [count]); // ← countが変更されたときだけ、この関数は新しく作り直される
  // 依存配列が空だと、この関数は初回レンダリング時のみ生成される
  
```

useCallbackを使うと何が起きるか？
初回レンダリング時: useCallbackは渡された関数を実行し、その関数インスタンスを記憶します。
再レンダリング時:
依存配列の値が変わっていない場合: useCallbackは新しく関数を作らず、記憶しておいた関数インスタンスを返します。
依存配列のいずれかの値が変わった場合: useCallbackは再度関数を実行し、新しく作られた関数インスタンスを記憶し、それを返します。


useCallbackの利点
パフォーマンス最適化:



子コンポーネントがReact.memoで最適化されていても（propsが変更されない限り再レンダリングしないようにする仕組み）、親から渡される関数が毎回新しいものだと、子コンポーネントは「propsが変わった！」と認識してしまい、不要な再レンダリングが発生してしまいます。
```tsx
// 子コンポーネント (React.memoで最適化)
const ChildComponent = React.memo(({ onAction }) => {
  console.log("ChildComponentがレンダリングされました");
  return <button onClick={onAction}>Do Action</button>;
});
```


要するにどういうこと？
useCallback(fn, deps) は、deps（依存配列）が変わらない限り、fn（関数）を再生成しないようにするフックです。
これにより、関数をpropsとして渡す子コンポーネントが、親の再レンダリングのたびに不必要に再レンダリングされるのを防ぐことができます（React.memoと併用時）。
「関数のアドレスを固定する」ようなイメージです。
いつ使うべき？
React.memoで最適化された子コンポーネントに関数を渡すとき。 これが最も一般的なユースケースです。



<a id="TSの型"></a>
## TSの型

TypeScriptの基本的な型一覧
```ts
// 文字列
let name: string = "taro";

// すべての数値。整数も少数も全部これ
let age: number = 30;

// 真偽値
let isStudent: boolean = true;

// オブジェクト
let person: { name: string; age: number } = {
  name: "hanako",
  age: 25,
};

// 配列
let numbers: number[] = [1, 2, 3, 4, 5];
let fruits: string[] = ["apple", "banana", "cherry"];

// タプル
let person: [string, number] = ["hanako", 25];

// Union型
let value: string | number;
value = "Hello";
value = 42;

// Literal型
let status: "success" | "error";
status = "success";
status = "error";

// function型
let add: (x: number, y: number) => number;
add = (a, b) => a + b;
// 関数の引数がオブジェクトの場合
let greet: (person: { name: string; age: number }) => string;
greet = (person) => `私の名前は ${person.name} です。年齢は ${person.age} です`;

// void型。関数が何も返さないことを表す。主に関数の戻り値の型として使用。
function logMessage(message: string): void {
  console.log(message);
}

// never型。絶対に値を持たない型。通常は無限ループや例外をスローする関数の戻り値の型として使用。
function throwError(message: string): never {
  throw new Error(message);
}
function infiniteLoop(): never {
  while (true) {
    // 無限ループ
  }
}
```


型エイリアス
```ts
// 型エイリアスを使用して、あらゆる型に名前を付けることができる。エイリアスを使用すると、エイリアスされた型を記述した場合と全く同じになる。

type Point = {
  x: number;
  y: number;
};
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

type ID = number | string;
```




<a id="shadcn"></a>
## shadcn

コンポーネントをインストールするコマンド  
例：  
`npx shadcn@latest add textarea`
