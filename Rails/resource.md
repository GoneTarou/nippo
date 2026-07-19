## resource（ルーティング）
Rails の resource は、RESTful なルーティングを自動生成してくれる便利な機能 
```ruby
resources :articles
```
resources :articles
これだけで、以下の7つのルートが自動生成されます：
| HTTPメソッド | パス | 説明 |
|-------------|------|------|
| GET | /articles | 一覧表示（index） |
| GET | /articles/new | 新規作成フォーム（new） |
| POST | /articles | 新規作成処理（create） |
| GET | /articles/:id | 詳細表示（show） |
| GET | /articles/:id/edit | 編集フォーム（edit） |
| PATCH | /articles/:id | 更新処理（update） |
| DELETE | /articles/:id | 削除処理（destroy） |

resource（単数形） - 単一リソース
```ruby
resource :profile
```
ID が不要な単一のリソース に使います。例えば、ログインユーザーのプロフィールなど、「自分のもの」しか扱わない場合です：
| HTTPメソッド | パス | 説明 |
|-------------|------|------|
| GET | /profile/new | 新規作成フォーム（new） |
| POST | /profile | 新規作成処理（create） |
| GET | /profile | 詳細表示（show） |
| GET | /profile/edit | 編集フォーム（edit） |
| PATCH | /profile | 更新処理（update） |
| DELETE | /profile | 削除処理（destroy） |

### ネストしたリソース
Railsのネストしたリソース（Nested Resources）とは、

「あるリソースが別のリソースに属していること」をルーティングで表現する機能です。

例えば日記アプリで、日記コメント（Comment）がある場合、コメントは日記に属するのでネストします。

```ruby
resources :articles do
  resources :comments  # /articles/:article_id/comments
end
```
| HTTPメソッド | パス | 説明 |
|-------------|------|------|
| GET | /articles/:article_id/comments | コメント一覧（index） |
| GET | /articles/:article_id/comments/new | 新規作成フォーム（new） |
| POST | /articles/:article_id/comments | 新規作成処理（create） |
| GET | /articles/:article_id/comments/:id | コメント詳細（show） |
| GET | /articles/:article_id/comments/:id/edit | 編集フォーム（edit） |
| PATCH | /articles/:article_id/comments/:id | 更新処理（update） |
| DELETE | /articles/:article_id/comments/:id | 削除処理（destroy） |

ネストしない場合
```ruby
resources :diaries
resources :comments
```
ルーティングは
```ruby
GET    /comments
POST   /comments
GET    /comments/:id
```
しかし、「どの日記のコメントなの？」という情報がURLから分かりません。
ネストする場合
```ruby
resources :diaries do
  resources :comments
end
```
コメント作成では、
```ruby
GET    /diaries/:diary_id/comments
POST   /diaries/:diary_id/comments
GET    /diaries/:diary_id/comments/:id
```
/diaries/5/commentsなら「日記ID=5のコメント」
```ruby
def create
  @diary = Diary.find(params[:diary_id])
  @comment = @diary.comments.build(comment_params)

  if @comment.save
    ...
  end
end
```
params は
```ruby
params[:diary_id]
#=> 5
```
### shallow routing
```ruby
resources :diaries do
  resources :comments, shallow: true
end
```
作成系
```ruby
GET  /diaries/:diary_id/comments
POST /diaries/:diary_id/comments
```
編集・削除系
```ruby
GET    /comments/:id
PATCH  /comments/:id
DELETE /comments/:id
```
どういうときにネストする？  
日記 → コメント  
投稿 → いいね  
記事 → タグ付け 
### 判断基準
「子リソースを操作するのに、親リソースのIDが必須か？」  
はい → ネストを検討する  
いいえ → ネストしない（または shallow: true を使う）
