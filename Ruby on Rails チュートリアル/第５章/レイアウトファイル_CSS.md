#　CSS
- CSSとは「Cascading Style Sheets」の頭文字を取ったもので、スタイルシートとも呼ばれる
- CSSはHTMLで作られた文書構造にデザインを加えて見栄えを整える役割を担っている
- CSSは以下のように書く。セレクター（p）でスタイルを設定するHTMLの要素を指定し、プロパティ（color）でスタイルづけするための方法を指定、プロパティ値（red）で具体的なスタイルをしている  
```
 p{
   color: red;
 }
 ```
 # Bootstrap
 - Twitter社が開発したCSSのフレームワーク
 - フォーム、ボタン、メニューといったWebページでよく使われる部品がテンプレートとして用意されている
  - 例えばdivタグのクラスとして"jumbotron"と指定すればテンプレートのトップ見出しを勝手に作ってくれる
 - Bootstrapを使うことでアプリケーションをレスポンシブデザインにできる
 - ```bootstrap-sass```gemを使ってRailsアプリケーションに導入できる
 - 導入の手順
  - Gemfileにbootstrap-sassを追加し、bundle installを実行してインストール
  - app/assets/stylesheets/ディレクトリに、costom.scssというファイルを作成
  - ファイルを作成したら、@importを使ってBootstrapを読み込む  
  ```
  @import "bootstrap-sprockets";
  @import "bootstrap";
  ```
# カスタムCSS
- Bootstrapだけでは微妙な部分もあるため、スタイルを与えるためのCSSを追加
- クラスに対してスタイルを適用するときは冒頭に.（ドット）をつける
- idに対してスタイルを適用するときは冒頭に#をつける
