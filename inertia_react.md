# Inertia+Reactの覚書

* 目次
    * [はじめに](#はじめに)



<a id="はじめに"></a>
## はじめに


PHPとComposerをインストール
Composer経由でLaravelインストーラーをインストール
`composer global require laravel/installer`

Laravelの新規プロジェクト作成
`laravel new example-app`
starter kitを聞かれたらReactを選択します
認証方法を聞かれたらLaravel's build-in authenticationを選択します
テストに使うフレームワークを聞かれたらPestを選択
`npm install`と`npm run build`するか聞かれたらyes


構成
resources/ts/Pages:
Laravelのコントローラーから Inertia::render() で指定されるフルページコンポーネント（.tsx ファイル）を格納します。
例: Users/Index.tsx, Users/Show.tsx, Dashboard.tsx, Auth/Login.tsx
resources/ts/Components:
再利用可能な部分的なUIコンポーネント（入れ子になるコンポーネント、.tsx ファイル）を格納します。
例: Button.tsx, Modal.tsx, TextInput.tsx, UserAvatar.tsx
resources/ts/Layouts (もし使う場合):
アプリケーション全体で共有される永続レイアウトコンポーネント（.tsx ファイル）を格納します。
例: AuthenticatedLayout.tsx, GuestLayout.tsx



Inertia + React 環境では、Reactコンポーネントは手動で作成するのが標準的な方法


フルページコンポーネントを作成がInertiaの基本的なアプローチです。
各Laravelルートが特定のReactコンポーネントに対応する

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
```jsx
// resources/js/Pages/Users/Index.jsx
import React from 'react';
import Layout from '@/Layouts/Layout'; // 例: 共通レイアウト

export default function Index({ users }) { // コントローラーから渡された 'users' プロップスを受け取る
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



Laravel ルーティング: 通常通り、routes/web.php などでルートを定義します。
Laravel コントローラー:
リクエストを処理し、必要なデータを準備します。
return Inertia::render() を使ってレスポンスを返します。このメソッドの第一引数には、表示したいReactコンポーネントの名前を指定し、第二引数にはそのコンポーネントに渡したい**データ（プロップス）**を連想配列で指定します。

フルページコンポーネント内でUIを部分的なコンポーネントに分割し、入れ子にすることは、コードを整理し、保守性や再利用性を高めるための標準的で推奨されるプラクティス。


Reactコンポーネントをアロー関数で書く








return Inertia::render('Users/Index', [
    'foo' => $foo,
    'bar' => $bar,
    'baz' => $baz,
]);



import React from 'react';

// 引数の部分で直接キー名を指定して受け取る
const UsersIndex = ({ foo, bar, baz }) => {
  return (
    <div>
      {/* 直接変数名でアクセスできる */}
      <div>Hello, {foo}!</div>
      <div>Hello, {bar}!</div>
      <div>Hello, {baz}!</div>
    </div>
  );
};

export default UsersIndex;


フルページコンポーネントはresources/js/Pages内に、
入れ子のコンポーネントは resources/js/Components内に、
格納するのですか？