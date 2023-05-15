# ユーザー一覧ページの作成
- ログインしているユーザーだけが表示できるようにしたい
- まずはログインしない状態でusersページにアクセスしようとし、ログインページにリダイレクトされることを確認するテストを書く
- 次にusersコントローラにおいて、beforeフィルターのlogged_in_userにindexアクションを追加して、indexアクションが実行される前にログインされているか確認するようにする
- indexアクションの中身として、@users = User.all　で全てのユーザーを取り出す（インスタンス変数@usersに代入することでビューでも使える）
- ビューでは、eachメソッドを使い、それぞれのユーザーの画像と、ユーザーページへのリンクがリスト形式で表示されるようにする 
  ```
  <% provide(:title, 'All users') %>
  <h1>All users</h1>

  <ul class="users">
    <% @users.each do |user| %>
      <li>
        <%= gravatar_for user, size: 50 %>
        <%= link_to user.name, user %>
      </li>
    <% end %>
  </ul>
  ```
# サンプルのユーザー
- gemfileにFaker gemを追加してbundle installする
- db/seeds.rbというファイルを編集してサンプルユーザーをデータベース上に生成する
```
# メインのサンプルユーザーを1人作成する
User.create!(name:  "Example User",
             email: "example@railstutorial.org",
             password:              "foobar",
             password_confirmation: "foobar")

# 追加のユーザーをまとめて生成する
99.times do |n|
  name  = Faker::Name.name
  email = "example-#{n+1}@railstutorial.org"
  password = "password"
  User.create!(name:  name,
               email: email,
               password:              password,
               password_confirmation: password)
end
```
- 一度データベースをリセットして（rails db:migrate:reset)、db/seeds.rbに追加したタスクを実行する（rails db:seed)
  - これでサンプルユーザーが100人生成される
# ページネーション
- 1ページに30人だけユーザーを表示するようにしたい
- gemfileにwill_pagenateとbootstrap-will_pagenateを追加してbundle install
- indexページのビューで、usersに関する部分を<%= will_pagenate %>ではさむ
- usersコントローラのindexアクションを定義する部分を、@users = User.allから　@users = User.paginate(page: params[:page])に変更することで
ページネーションが行われるようになる
- fixtureに30人のユーザーを追加して、paginationクラスを持ったdivタグが存在するかの確認とユーザーページに飛べるユーザーの名前が表示されているか確認するテストを書く
