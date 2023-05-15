# PasswordResetsコントローラ
- パスワード再設定用のコントローラを作る  
```rails generate controller PasswordResets new edit --no-test-framework```
- パスワード再設定のためのフォームと、パスワード変更のためのフォームが必要なので、new、create、edit、updateのルーティングも用意する
# 新しいパスワードの設定
- 記憶トークンや有効化トークンの時と同様、トークン用の仮想的な属性とそれに対応するダイジェストを用意する
- usersテーブルにreset_diges属性tとreset_sent_at属性をを追加するために以下を実行
```
rails generate migration add_reset_to_users reset_digest:string \
> reset_sent_at:datetime
```
- マイグレーションを実行
- form_withを使ってログインフォームとパスワード再設定のビューを作る
# createアクションでパスワード再設定
- 作成したパスワード再設定のためのフォームから送られたメールアドレスをキーとしてユーザーをデータベースから見つけ、パスワード再設定用トークンと送信時のタイムスタンプでデータベースの属性を更新
```
def create
    @user = User.find_by(email: params[:password_reset][:email].downcase) # emailをキーに探し出したユーザーを@userに代入
    if @user  # 送られたemailを持つユーザーが存在する場合
      @user.create_reset_digest # パスワード再設定用トークンを作成してreset_digestとreset_sent_at属性を更新する（modelでメソッドを定義）
      @user.send_password_reset_email # メールを送信（modelでメソッドを定義）
      flash[:info] = "Email sent with password reset instructions"  # メッセージを表示
      redirect_to root_url  # ルートURLにリダイレクト
    else  # 送られたemailを持つユーザーが存在しない場合
      flash.now[:danger] = "Email address not found"  # 警告メッセージを表示
      render 'new'  # フォーム画面を表示
    end
 end
 ```
- Userモデルにパスワード再設定用メソッドを追加
  - パスワード再設定の属性を設定する（新しいトークンをreset_token属性と定義、
  それをダイジェスト化したreset_digest属性、メソッドが実行されたreset_sent_at属性をユーザーに追加する)create_reset_digestメソッドを定義
  - パスワード再設定のメールを送信するメソッド
  - User.reset_tokenを使えるようにするためにattr_accessorを使う
