## チュートリアル
## Dockerデスクトップのインストール
- Dockerデスクトップはコンテナ化されたアプリケーションとマイクロサービスを構築・共有するためのツール  
``` $ brew cask install docker```でインストール

## Clone
- リポジトリをクローンする  
`$ git clone https://github.com/docker/getting-started.git`
- ここにはイメージを作成してコンテナとして実行するために必要なものが全て含まれている

## Build
- 次にイメージを作成する
- Dockerイメージはコンテナのためのプライベートなファイルシステムで、コンテナが必要とするすべてのファイルとコードを提供する  
```
$ cd getting-started
$ docker build -t docker101tutorial
```
## Run
- コンテナを実行する
- コンテナはイメージを元にしたもので、コンテナを起動するとPCの他の場所から安全に隔離されたリソースを使用してアプリケーションを起動できる  
`$ docker run -d -p 80:80 --name docker-tutorial docker101tutorial`
## Share
- イメージを保存して共有する
- イメージをDocker Hubに保存・共有すると他のユーザーがどんなマシンでも、簡単にイメージをダウンロードして起動できるようになる  
```
$ docker tag docker101tutorial michinosuke/docker101tutorial
$ docker push michinosuke/docker101tutorial
```
## 実際によく使うコマンド
- `git clone`でリポジトリをコピー
`git clone https://github.com/xxxxx`
- cd {プロジェクトのディレクトリ}
- Docker　Composeを使ってアプリに必要なコンテナを起動する  
`docker-compose up -d app web mysql`
- 必要なパッケージのインストールなどを行う  
`make setup`
