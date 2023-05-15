# チェックボックスの追加
- ログインフォームにチェックボックスを追加する
- これだけで、ログインフォームから送信されるparamsハッシュにはチェックボックスの値が含まれ、オンの時は１、オフの時は２になる
- paramsハッシュのこの値が１であればremember userを、0であればforget userが実行されるようにする
  - createメソッドに ```params[:session][:remember_me] == '1' ? remember(user) : forget(user)``` を追加
- これでログインシステムの実装完了
# テストの作成
- チェックボックスのテスト
  - テストユーザーとしてログインするメソッドをテストヘルパーに追加
  - remember_meが1の時、cookiesのremeber_tokenが空ではないことを確認
  - remember_meが１の状態で（永続ログインを実行中）にlogout_pathにDELETEリクエストを送り、ログアウトする
  - その後remember_meが０の時、cookiesのremember_tokenが空であることを確認
- Remember me　のテスト
  -　fixtureでuser変数を定義する
  -　渡されたユーザーをrememberメソッドで記憶する
  -　current_userが、渡されたユーザーと同じであることを確認する 
