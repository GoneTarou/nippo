## Rails8でrails g authenticationを実行後勝手にログインが必須にされる
https://railsguides.jp/security.html　

bin/rails generate authentication実行後、app/controllers/concerns/authentication.rbにある

```ruby
included do
    before_action :require_authentication
    helper_method :authenticated?
  end
```
によって、すべてのページにログインが要求される。ログインしていない場合はsessions/newに遷移される。

### included do とは
included do は、モジュールが他のクラスに include されたときに自動的に実行される処理 を定義するためのフックメソッドです。

このコードは以下のような意味になります：

- included do ：このモジュールが include されたときに実行される処理を定義
- before_action :require_authentication ：include 先のコントローラに before_action を追加
- helper_method :authenticated? ：authenticated? メソッドをビューでも使えるようにする

```ruby
class_methods do
  def allow_unauthenticated_access(**options)
    skip_before_action :require_authentication, **options
  end
end
```

allow_unauthenticated_access only: %i[ new create ]

newとcreateメソッドの時はログインなしでアクセスできるようにする

```ruby
def authenticated?
  resume_session
end

```ruby
def require_authentication
  resume_session || request_authentication
end
```
- resume_session でセッションを復元しようとする
- 成功すれば認証OK
- 失敗すれば request_authentication でログインページへリダイレクト

```ruby
def resume_session
  Current.session ||= find_session_by_cookie
end

def find_session_by_cookie
  Session.find_by(id: cookies.signed[:session_id]) if cookies.signed[:session_id]
end
```
- Current.session ：リクエストごとに保持される現在のセッション情報
- cookies.signed[:session_id] ：暗号化された Cookie からセッション ID を取得
- Session.find_by ：データベースからセッション情報を取得

```ruby
def request_authentication
  session[:return_to_after_authenticating] = request.url
  redirect_to new_session_path
end

def after_authentication_url
  session.delete(:return_to_after_authenticating) || root_url
end
```
- ログイン前にアクセスしようとした URL を覚えておく
- ログイン後、元々アクセスしたかったページに戻れる

具体例
  ```ruby
  # SessionsController

def create
  if user = User.authenticate_by(email: params[:email], password: params[:password])
    start_new_session_for(user)
    redirect_to after_authentication_url  # ← ここで元のページに戻る
  else
    render :new, status: :unprocessable_entity
  end
end
```
```ruby
def start_new_session_for(user)
  user.sessions.create!(user_agent: request.user_agent, ip_address: request.remote_ip).tap do |session|
    Current.session = session
    cookies.signed.permanent[:session_id] = { value: session.id, httponly: true, same_site: :lax }
  end
end
```
- user.sessions.create! ：データベースに新しいセッションレコードを作成
- user_agent と ip_address ：セキュリティのために記録
- cookies.signed.permanent ：暗号化された永続 Cookie を設定
- httponly: true ：JavaScript からアクセス不可（XSS 対策）
- same_site: :lax ：CSRF 対策

```ruby
def terminate_session
  Current.session.destroy
  cookies.delete(:session_id)
end
```
- ログアウト処理

