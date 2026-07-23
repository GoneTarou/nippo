## Rails 8: strong parametersの新しいparams.expectの使い方
https://railsguides.jp/action_controller_overview.html

https://techracho.bpsinc.jp/hachi8833/2024_11_12/146280

params.expect は、Rails 8で追加されたStrong Parametersの新しい書き方です。
```ruby
params.require(:user).permit(:name, :email)
```
を、よりシンプルかつ安全に書けるようになりました。

従来の書き方ではフォームが:nameと:emailなら
```ruby
def user_params
  params.require(:user).permit(:name, :email)
end
```
これでpermitはnameとemailだけ許可

### Rails8では
```ruby
def user_params
  params.expect(user: [:name, :email])
end
```
だけでよくなった。

user.nameやuser.emailが期待どおりでない場合は、期待した構造ではないとしてActionController::ParameterMissingが発生します。  
そのため、「このパラメータ構造で送られてくるはず」という前提をより厳密にチェックできます。

expectの場合、ActionController::ParameterMissingが良いのは、「リクエスト自体がおかしいことを早い段階で教えてくれる」からです。これを「Fail Fast（早く失敗する）」と呼ぶ

### permitの場合
会員登録で
```
def user_params
  params.permit(:username, :password)
end
```
本来は、
```JSON
{
  "username": "alice",
  "password": "secret"
}
```
が送られるはずなのに、フロントエンドのバグで{}が送られてきた場合、
User.create!(user_params)すると、Username can't be blank　Password can't be blankとなり
パラメータが送られていないのか、バリデーションに失敗したのかが分かりづらいです。

expectなら{}が送られると、「そもそもリクエストの形式が間違ってる」のでActionController::ParameterMissingで例外発生となる。
つまり、

400 Bad Request
リクエストの形式がおかしいのと、

422 Unprocessable Entity
リクエストの形式は正しいが、内容が不正（バリデーションエラーを区別できます。

ユーザー登録なら
```ruby
def user_params
  params.expect(
    user: [
      :username,
      :password,
      :password_confirmation
    ]
  )
end
```
ログインなら
```ruby
username, password = params.expect(
  :username,
  :password
)
```

### Railsがexpectを追加した理由

「userの中にusernameとpasswordがあることを前提にこのアクションを書いています」

という開発者の意図をコードで表現しています。

その前提が崩れたら、処理を続けるのではなく即座に例外を出して止める方が、安全で原因も分かりやすいという考え方です。
