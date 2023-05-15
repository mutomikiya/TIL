# 基本的なモデル
- Micropostモデルは、マイクロポストの内容を保存するcontent属性と特定のユーザーとマイクロポストを関連づけるuser_id属性の２つの属性だけを持つ
- マイクロポストの投稿にはString型ではなくText型を使う
  - ある程度の量のテキストを格納するときにはText型を使う
  - String型は255文字まで格納できるがText型の方が表現豊かなマイクロポストを実現できる
  - 例えば言語に応じて投稿の長さを調節するなど
- generate modelコマンドでMicropostモデルを生成する  
```rails generate model Micropost content:text user:references```
  - reference型を利用することで、自動的にインデックスと外部キー参照付きのuser_idカラムが追加され、UserとMicropostモデルを関連付けする下準備をしてくれる
  - マイグレーションをしてデータベースを更新する
- バリデーションの追加
  - user_idの存在性のバリデーションに対するテストを追加する
  - モデルにマイクロポストのuser_idに対するバリデーションを追加する```validates :user_id, presence: true```
  - Micropostに存在性と文字の長さについてのバリデーションに対するテストを追加する
  - モデルにバリデーションを追加する```validates :content, presence: true, length: { maximum: 140 }```
- User/Micropostの関連付け
  - それぞれのマイクロポストは1人のユーザーと関連づけられ、それぞれのユーザーは複数のマイクロポストと関連付けられる
  - belong_to/has_many関連付けを行うことで、micropost.userやuser.microposts、user.microposts.create(arg）などのメソッドが使える
  - micropostモデルにbegongs_to :user、userモデルにhas_many :micropostsを追加
- 最新のマイクロポストを最初に表示する
  - まずテストを書く```assert_equal microposts(:most_recent), Micropost.first```
  - fixtureも定義しておく
  - データベースから要素を取得したときのデフォルトの順序を指定するメソッドdefault_scopeを使って順序を指定する  
  ```default_scope -> {order(created_at: :desc)}```
- ユーザーと同時にマイクロポストも削除されるようにする
  - has_manyメソッドにオプションを渡すことで実装できる  
  ```has_many :microposts, dependent::destroy``` 
  - ユーザーと一人削除してみて、マイクロポストの数が減っているかテストを書く
