# Webサーバー
- Pumaは多数のリクエストを捌くことに適したRuby/Rackアプリケーション用のサーバー
- Procfileと呼ばれる、Heroku上でPumaのプロセスを走らせる設定ファイルを作成する
# データベースの設定
- 本番環境で使うデータベースはPostgreSQL
- データベース設定ファイルconfig/database.ymlのprudctionセクションを更新するだけ
