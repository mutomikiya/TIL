# リソース
- リソースは、リレーショナルデータベースの作成・取得・更新・削除操作と、４つの基本的なHTTP requestメソッド（POST,GET,PATCH,DELETE)の両方に対応している
- リソースへの参照はリソース名とユニークなIDを使う
- ユーザーをリソースとみなす場合id = 1のユーザーを参照するということは、/users/1というURLに対してGETリクエストを発行するということを意味する
- REST機能が有効になっているとGERリクエストは自動的にshowアクションとして扱われる
# リソースの有効化
- routesファイルにresources :usersというコードを追加する
  - これによってユーザー情報を表示するURL（/users/1）が有効になる
  - また、ユーザーのURLを生成するための名前付きルートとともに、RESTfulなUsersリソースで必要となる全てのアクションが利用できるようになる
# ユーザーのページを表示する
- 実際にページを表示するためにはビューを作成する必要がある
  - touchコマンドなどを使ってapp/view/users/show.html.erbファイルを作成
- ユーザー表示ビューを正常に動作させるために、Usersコントローラないのshowアクションに対応する@user変数を定義する必要がある
  -　findメソッドを使ってデータベースからユーザーを取り出す  
  ```
  def show
    @user = User.find(params[:id])
  end
  ```
- Gravatarを導入したりSCSSを使ったりしてユーザー表示ページにスタイルを与える
