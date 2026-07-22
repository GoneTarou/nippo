## resources :users

resources :users と書くと、7つのルート（URL）が自動的に作られます 。

| HTTPメソッド | URL | アクション | 用途 |
|------------|-----|----------|------|
| GET | /users | index | ユーザー一覧表示 |
| GET | /users/new | new | 新規登録フォーム表示 |
| POST | /users | create | ユーザー作成 |
| GET | /users/:id | show | ユーザー詳細表示 |
| GET | /users/:id/edit | edit | 編集フォーム表示 |
| PATCH/PUT | /users/:id | update | ユーザー情報更新 |
| DELETE | /users/:id | destroy | ユーザー削除 |

:id の部分について

:id は 実際のユーザーIDに置き換わる部分 です。例えば：

/users/1 → IDが1のユーザーの詳細
/users/5 → IDが5のユーザーの詳細
/users/100/edit → IDが100のユーザーの編集画面
