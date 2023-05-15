# 本番環境でのSSL
- SSLは、WebサーバとWebブラウザとの通信においてやり取りされるデータの暗号化を実現する技術
- Railsでは、SSLはproduction.rbという本番環境の設定ファイルを１行修正するだけで済む
  - ```config.force_ssl = true```と加える
- 普通はドメインごとにSSL証明書を購入しセットアップする必要があるが、Heroku上でサンプルアプリケーションを動かす上ではHerokuのSSL証明書に便乗できる
