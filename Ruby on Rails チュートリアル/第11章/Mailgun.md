# 本番環境でのメール送信
- MailgunというHerokuアドオンを利用する
- ```config/environments/production.rb```に以下のような設定を追加
```
  config.action_mailer.raise_delivery_errors = true
  config.action_mailer.delivery_method = :smtp
  host = '<あなたのHerokuサブドメイン名>.herokuapp.com'
  config.action_mailer.default_url_options = { host: host }
  ActionMailer::Base.smtp_settings = {
    :port           => ENV['MAILGUN_SMTP_PORT'],
    :address        => ENV['MAILGUN_SMTP_SERVER'],
    :user_name      => ENV['MAILGUN_SMTP_LOGIN'],
    :password       => ENV['MAILGUN_SMTP_PASSWORD'],
    :domain         => host,
    :authentication => :plain,
  }
 ```
 - Herokuにデプロイする```git push heroku```
 - migrationファイルの変更を反映させる```heroku run db:migrate```
 - Herokuアドオンを追加するために```heroku addons:create mailgun:starter```コマンドを実行
 - ```heroku addons:open mailgun```でMailgunダッシュボードを開き、受信するメールアドレスを認証する
 - ここまでで準備は完了、あとは本番環境で新しくユーザー登録を行うと、登録したメールアドレスに実際にメールが送られて、ユーザーを有効化できる
