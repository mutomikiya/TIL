# Guardとは
- Guardはファイルシステムの変更を監視し、ファイルが変更されると自動でテストを実行してくれるツール
- まずGemfileでguard gemをアプリケーション内に取り込む
- guardfileを編集して適切なテストが実行されるようにする  
```guard :minitest, spring: "bin/rails test", all_on_start: false do```
- 以下のコマンドを実行してプロジェクト内の全ファイルをGuardで監視できるようにする
```
$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
```
- これで設定は完了で、新しいターミナルを開いて次のコマンドを実行する  
```$ bundle exec guard```
- この状態で、何かのファイルを変更すると自動でテストしてくれる
