- [1. 使用する`docker`のベースイメージを選定する](#1-%e4%bd%bf%e7%94%a8%e3%81%99%e3%82%8bdocker%e3%81%ae%e3%83%99%e3%83%bc%e3%82%b9%e3%82%a4%e3%83%a1%e3%83%bc%e3%82%b8%e3%82%92%e9%81%b8%e5%ae%9a%e3%81%99%e3%82%8b)
- [2. `Dockerfile`を記述する](#2-dockerfile%e3%82%92%e8%a8%98%e8%bf%b0%e3%81%99%e3%82%8b)
- [3. `Dockerfile`をイメージとしてビルドする](#3-dockerfile%e3%82%92%e3%82%a4%e3%83%a1%e3%83%bc%e3%82%b8%e3%81%a8%e3%81%97%e3%81%a6%e3%83%93%e3%83%ab%e3%83%89%e3%81%99%e3%82%8b)
- [4. `docker-compose`を利用してコンテナを実行する](#4-docker-compose%e3%82%92%e5%88%a9%e7%94%a8%e3%81%97%e3%81%a6%e3%82%b3%e3%83%b3%e3%83%86%e3%83%8a%e3%82%92%e5%ae%9f%e8%a1%8c%e3%81%99%e3%82%8b)
- [5. リファレンス](#5-%e3%83%aa%e3%83%95%e3%82%a1%e3%83%ac%e3%83%b3%e3%82%b9)
- [6. 補足と実例](#6-%e8%a3%9c%e8%b6%b3%e3%81%a8%e5%ae%9f%e4%be%8b)

# 1. 使用する`docker`のベースイメージを選定する

- `MySQL`,`nginx`や`apache`など、公式からイメージが提供されているものも数多く存在します。
  - 詳細については、`docker hub`で個々のイメージのドキュメントを確認してください。
    - https://hub.docker.com/_/httpd
    - https://hub.docker.com/_/nginx
    - https://hub.docker.com/_/mysql
- もし提供されたイメージの状態そのままでよいなら、`docker run -d`を実行してそのまま実行します。
  - 多くの場合はそうではないので、以降を記載します。

# 2. `Dockerfile`を記述する

- 任意のディレクトリに`Dockerfile`と命名したファイルを作成します。
- `FROM` 文でベースイメージを指定します。
- `RUN` 文で、コンテナ起動時のセットアップを行います。
  - サーバ構築時に行う`apt-get update`などの手順はここに該当します。
- 例として、`ubuntu` イメージを利用して `nginx` を導入したイメージを作成したいとするなら、`Dockerfile`に以下のように記載します。

  - ```dockerfile
        FROM ubuntu:14.04
        RUN apt-get update -y
        RUN apt-get install -y -q nginx
    ```
    - 複数記載できることを示すために`RUN`を分けていますが、不要なら`&&`を用いてワンライナーにまとめることを推奨します。
      - 特に`RUN apt-get update`後に別個に`RUN`を実行すると、キャッシュの関係で予期しない動作をすることがあります。
    - 後述する`compose`を利用しないならば、`EXPOSE`文を用いたポート指定や、`VOLUME`文を用いたマウント指定を記載します。
    - 参考：http://blog.flup.jp/2016/02/12/devenv_nginx_with_docker/
    - このリポジトリの`dockerfile-example`ディレクトリに実例としてファイルが配置してあります。
  - もちろんですが、公式`nginx`イメージを利用すればいい話なので、あくまでも一例です。

# 3. `Dockerfile`をイメージとしてビルドする

- 順を追って理解したい人のために記載しますが、最低限でいいという場合はこの項目は無視して構いません。
  - 実際には`docker-compose`をはじめとした複数コンテナが扱える手段を利用するのがほとんどだからです。
- Dockerfile の存在するディレクトリで、コマンドプロンプトなどを利用して以下のコマンドを実行します。
  - ```
    docker build -t mycontainer/nginx:latest .
    ```
  - `-t`オプションで、`name:tag`形式のタグを付与できます。
    - 上記の例では、先ほど作成した`Dockerfile`をもとに`latest`タグのついた`mycontainer/nginx`イメージをビルドします。
  - ビルド後、`docker images`を実行すると、ローカルに存在するイメージの中に、ビルドしたイメージが追加されていることが確認できます。
    - `docker run`コマンドを利用して、作成したイメージをコンテナとして実行できます。

# 4. `docker-compose`を利用してコンテナを実行する

- 実例は、このリポジトリの`docker-compose-example`ディレクトリを参考にしてください。
- `docker-compose.yml`ファイルを作成します。
  - もし`2.`で`Dockerfile`を作成したなら、削除しても構いません。
  - `docker-compose.yml`には以下の通り記載します。
    - ```yaml
      version: "3.7"
      services:
        web_server:
          container_name: my-nginx
          build:
            context: ./nginx
          volumes:
            - ./src:/usr/share/nginx/html
          ports:
            - "8085:80"
      ```
      - `web_server:` このコンテナに付与されたサービス名です。
      - `container_name:` このコンテナは、`my_nginx`と命名されます。
      - `build:` このコンテナは、`nginx`ディレクトリ以下から`Dockerfile`を探査してコンテナを作成します。
      - `volumes:` このコンテナは、コンテナの`usr/share/nginx/html`配下に`src`ディレクトリをマウントします。
      - `ports:` ポート転送として、コンテナ内部の`80`番ポートを、`Docker`を実行しているホストマシンの`8085`ポートに転送します。
- 上記の定義に基づいて、必要なファイルを配置します。

  - `nginx`ディレクトリを作成し、配下に新たな`Dockerfile`を作成します。

    - ```dockerfile
      FROM nginx:latest
      RUN apt-get update -y -q
      ```

  - `src`ディレクトリを作成し、`index.html`ファイルを作成します。
    - ```html
      <h1>Web server is working!</h1>
      ```
    - これは`nginx`がデフォルトで表示するページとなります。

- `docker-compose.yml`が配置されたディレクトリで、コマンドプロンプト等から`docker-compose up -d`を実行します。
  - （まだ存在しないなら）イメージのビルドと、コンテナの起動を同時に行います。
  - コマンドを実行したならコンテナが起動しているので、ブラウザから[http://localhost:8085](http://localhost:8085)にアクセスすると、`index.html`が表示されます。
- `docker-compose down`を実行すると、コンテナが終了します。（一時停止ではないので注意してください）
- `Dockerfile`などコンテナの定義を変更したなら、コンテナを終了した上で`docker-compose build`で再度ビルドを行えます。

# 5. リファレンス

- Dockerfile のベストプラクティス(有志訳)
  - http://docs.docker.jp/engine/articles/dockerfile_best-practice.html
- 効率的に安全な Dockerfile を作るには
  - https://qiita.com/pottava/items/452bf80e334bc1fee69a

# 6. 補足と実例

- `services` 定義は何のために存在するのか補足します。
  - `5.`でベストプラクティスとして紹介されている通り、一つのコンテナは一つのプロセスにのみ関心を持つのが理想的です。`services`定義を利用することで、関心の分離を実現できます。
  - つまり、アプリケーションコードを実行するコンテナや、それを外部へと公開するための`apache`などのウェブサーバ、`mysql`や`postgres`などのデータベース、`redis`などのミドルウェアのコンテナは、それぞれ別の`service`として定義し、各々のコンテナが各々のタスクを担当します。
  - 典型例として、有志が`github`で公開している`laravel`および`django`の`docker-compose.yml`を示します。
    - https://github.com/aschmelyun/docker-compose-laravel/blob/master/docker-compose.yml
    - https://github.com/realpython/dockerizing-django/blob/master/docker-compose.yml
