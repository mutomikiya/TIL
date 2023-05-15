## データモデルの作成
- user_followerテーブルとuser_followingテーブルをそれぞれ作ってuserテーブルとhas_manyで関連づけを行うと、無駄が多い
- また、この関係は一方向のものであって、左右対称ではない
- フォローでは新たに「関係」を作り、フォロー解除では「関係」を削除する、と考える
- 解決策として、relationshipsテーブルを作り、その属性としてid、follower_id、followed_idを設定する
  - これでuser1がuser2をフォローしたとき新たにrelationship1が作成され、その関係の内容はfollower:user2、followed:user1、というふうに表すことができる
  - フォロー解除の時はrelationship1が削除されたと考えることもできる
  - followe_idとfollowed_idの組はユニークになるように設定する  
  ```add_index :relationships, [:follower_id, :followed_id], unique: true```
  - 相互フォロー状態の時はfollowとfollowedが逆になっている２つのrelationshipが作られている状態を意味する
- userモデルとrelationshipsモデルを関連づける
  - userモデルを編集して、userからfollower_idに対してhas_manyの関連付けをする  
    ```
    has_many :active_relationships, class_name:  "Relationship", 
                                    foreign_key: "follower_id",
                                    dependent:   :destroy
    ```
  - relationshipモデルを編集して、followe/followedはいずれかのuserに属していることをbelongs_toを使って表す
  - followingの関連付けを追加する
   - この際、followedsという名前は不適切であるから代わりにfollowingという名前を使う  
   `has_many :following, through: :active_relationships, source: :followed`
 - フォロワーについては、フォローとほぼ同様
## フォローに関わるメソッドを定義する
- other_userをフォロー、フォロー解除するメソッドはそれぞれ次のように表せる  
  ```
  def follow(other_user)
    following << other_user
  end
  
  def unfollow(other_user)
    active_relationships.find_by(followed_id: other_user.id).destroy
  end
  ```
  
  
 
