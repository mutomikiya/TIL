# 記憶トークン作成の準備
- 記憶トークンを作成して、cookiesメソッドによる永続的cookiesの作成や、安全性の高い記憶ダイジェストによるトークン認証に利用する
  - 特定のユーザーと紐づいたトークンを作成して、cookiesでブラウザ側に永続的に保存、またデータベースにトークンに対応する記憶ダイジェストを保存しておく
  - ブラウザを再起動したとしても永続的cookiesはトークンを保持しているため、ログアウト操作を行わない限りずっとログイン状態が保たれる
- まずUserモデルにremember_digest属性を追加
- 記憶トークンにする長くてランダムな文字列の生成には、SecureRandomモジュールのurlsafe_base64メソッドを使う
- モデルファイルにランダムなトークンを返すメソッドを追加する（User.new_token)
# user.rememberメソッド
- 記憶トークンをユーザーと関連づけ、トークンに対応する記憶ダイジェストをデータベースに保存する、というメソッドを作成する
- パスワード実装における仮想のpassword属性のような、仮想のremember_token属性を作成するためにattr_accessorを使う
- attr_accessor :remember_tokenを追加してから、永続セッションのためにユーザーをデータベースに記憶するuser.rememberメソッドをUserモデルに追加する  
```
def remember
 self.remember_token = User.new_token   
 # new_tokenメソッドで返されるランダムなトークンをremember_token属性として設定する
 update_attribute(:remember_digest, User.digest(remember_token))   
 # remember_digest属性を、remember_tokenのハッシュ値に更新する（remember_tokenにdigestメソッドを適用してハッシュ値に変換している）
end
```
# 永続cookiesへの保存
- ``` cookies.permanent[:remember_token] = remember_token```で記憶トークンをブラウザの永続cookiesに保存できる
- ``` cookies[:user_id] = user.id```でユーザーIDをcookiesに保存できる
- ただ、これでは生のテキストとして保存されてしまうので、よくない
- ```cookies.signed[:user_id] = user.id```とすることでcookieをブラウザに保存する前に安全に暗号化できる
- cookiesを設定すると、以後のページのビューでは```User.find_by(id: cookies.signed[:user_id])```でユーザーを取り出せる
- 続いて、bcryptを使ってcookies[:remember_token]がremember_digestと一致することを確認し、一致したらtrueを返すメソッド（authenticated?(remember_token）を定義する   
```
def authenticated?(remember_token)
 BCrypt::Password.new(remember_digest).is_password?(remember_token)
end
```
# ログインしたユーザーを記憶する
- これでログインしたユーザーを記憶する準備が整ったため、createアクションの中にremember userを追加する
- そして、その中身をヘルパーで定義する  
  ```
  def remember(user)
    user.remember
    cookies.permanent.signed[:user_id] = user.id
    cookies.permanent[:remember_token] = user.remember_token
  end
  ```
- これで、createアクション（ログイン）した際にユーザーごとに記憶トークンが生成されてダイジェストがデータベースに保存される、その後ユーザーIDと記憶トークンが永続cookiesに保存される
- corrent_userヘルパーを修正  
```
  def current_user
    if (user_id = session[:user_id]) # （ユーザーIDにユーザーIDのセッションを代入した結果）ユーザーIDのセッションが存在すれば
      @current_user ||= User.find_by(id: user_id)　# そのユーザーIDを持つユーザーを取り出し@current_userとする
    elsif (user_id = cookies.signed[:user_id])　# （ユーザーIDに、cookiesに保存されたユーザーIDを代入した結果）ユーザーIDが存在すれば
      user = User.find_by(id: user_id)　# userにそのユーザーを代入する
      if user && user.authenticated?(cookies[:remember_token])　# そして、ユーザーが存在してかつcookiesのremember_tokenとユーザーのremember_digest属性が一致すれば
        log_in user　　# ユーザーをログインさせる
        @current_user = user　# そのユーザーを@current_userとする
      end
    end
  end
 ```
 # ユーザーを忘れる
 - ここまでで一時的なログイン（一時cookieにユーザーIDを保存、ブラウザ終了時に破棄）とログイン状態の保持（永続cookiesにユーザーIDと記憶トークンを半永久的に保存、cookiesの記憶トークンと一致するremember_digest属性を持つユーザーがいれば、そのユーザーとしてログインしていると扱う）の仕組みを整えたが、ログアウトの仕組みがない
 - そこでユーザーを忘れるためのメソッドを定義する
 - モデルファイルに、ユーザーのログイン情報を破棄するforgetメソッドを追加する  
 ```
  def forget
    update_attribute(:remember_digest, nil) # remember_digest属性をnilに更新する
  end
 ```
 - forgetメソッドを使って、forgetヘルパーメソッドとlog_outヘルパーメソッドを追加
 ```
  def forget(user)
    user.forget # ユーザーのremember_digest属性をnilにする
    cookies.delete(:user_id)　
    cookies.delete(:remember_token) # 永続cookiesにあるuser_idとremember_tokenを削除する
  end
  
   def log_out
    forget(current_user) # ログイン中のユーザーにforgetヘルパーメソッドを適用
    session.delete(:user_id)　# そのユーザーのセッションを削除する
    @current_user = nil　# @current_userをnilにする
  end
 ```
 - ログインしている時のみログアウトする、などの小さなバグの修正をする
