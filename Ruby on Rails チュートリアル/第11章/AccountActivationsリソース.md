# AccountActivationsコントローラ
- rails generateでAccountActivationsコントローラを作成
- ルーティングにアカウント有効化用のresources行を追加
``` resources :account_activations, only: [:edit]```
  - GETリクエストが/account_activation/トークン/editに送られたときにaccount_activationsコントローラのeditアクションを呼び出す
# AccountActivationのデータモデル
- データベースには有効化トークンをハッシュ化した文字列をおいておきたい
- Usersモデルにstringのactivation_digest属性、booleanのactivated属性、datetimeのactivated_at属性を加えるためのマイグレーションファイルを作成する  
```$ rails generate migration add_activation_to_users activation_digest:string activated:boolean activated_at:datetime```
  - activated属性のデフォルトの論理値はfalseにしておく
  - マイグレーションを実行する
  - これでデータベースのカラムに３つの属性が追加された
- ユーザーの新しい登録のために必要であるから、ユーザーオブジェクトが作成される前に有効化トークンや有効化ダイジェストを作成しておく必要がある
  - 有効化トークンとダイジェストを作成しそれを代入するcreate_activation_digestメソッドと、それを新しいユーザーのcreateアクションが行われる前に実行するための
  before_create:create_activation_digestというコールバックを設定する  
  ```
    def create_activation_digest
      self.activation_token  = User.new_token # ユーザーに紐づいた有効化トークンを作成
      self.activation_digest = User.digest(activation_token)  # 有効化トークンをハッシュ化した有効化ダイジェストを作成
    end
    ```
- テスト時のサンプルとユーザーを有効化しておく（activated: trueとactivated_at: Time.zone.nowを追加する）
  - データベースを初期化して、サンプルデータを再度生成し直す 
