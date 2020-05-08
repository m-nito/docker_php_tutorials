# 実用例から見る Dockerfile

- `東京都 新型コロナウイルス感染症対策サイト`の`Dockerfile`から実際の利用例を見てみます。

  - 東京都の`git`リポジトリには、以下からアクセスしてください。
  - https://github.com/tokyo-metropolitan-gov/covid19/

- `Dockerfile`を開くと、以下のようになっています。
- ```Dockerfile
    FROM node:10.19-alpine

    ENV PROJECT_ROOTDIR /app/

    WORKDIR $PROJECT_ROOTDIR

    COPY package.json yarn.lock $PROJECT_ROOTDIR

    RUN yarn install

    COPY . $PROJECT_ROOTDIR

    EXPOSE 3000
    ENV HOST 0.0.0.0

    CMD ["yarn", "dev"]
  ```

- 内容を解説します。

  - `FROM:`
    - ベースイメージとして`node`のイメージを利用しています。
  - `ENV:`
    - 環境変数として、`/app/`をプロジェクトディレクトリとしてセットしています。
  - `WORKDIR:`
    - 上記で指定した環境変数をもとに、ディレクトリを移動します。
  - `COPY package.json yarn.lock`
    - パッケージ管理用のファイルを、コンテナのプロジェクトディレクトリにコピーします。
    - 当該ファイルの中身は、実際のリポジトリから当該ファイルを参照してください。
  - `RUN yarn install`
    - 上記でコピーした`package.json yarn.lock`ファイルを利用して、依存ライブラリなどのパッケージを導入します。
    - 一般的なパッケージ管理の方法については、たとえば`PHP`であれば`composer`などが存在します。以下の記事を参照してください。
      - 基本的な解説(2016/04)
        - https://qiita.com/atwata/items/d6f1cf95ce96ebe58010
      - `Dockerfile`と連携した`composer`利用例
        - https://qiita.com/niisan-tokyo/items/8cccec88d45f38171c94
  - `COPY . $PROJECT_ROOTDIR`
    - ホストディレクトリの内容を、プロジェクトディレクトリにすべてコピーします。
    - この時、コンテナに導入する必要のないファイルは、`.dockerignore`ファイルに定義されています。
  - `EXPOSE 3000, ENV HOST 0.0.0.0`
    - ポート 3000 を解放し、`localhost:3000`などのようにホストからアクセスできるようにします。
  - `CMD ["yarn", "dev"]`
    - コンテナ内で`yarn`コマンドを実行し、アプリケーションを起動します。
    - ここでの`yarn`コマンドは、`package.json`で`dev`として指定された初期起動スクリプトを実行します。

- 上記のような処理を行うことで、コンテナを起動した開発者は、ただちにアプリケーションの開発/デバッグを行うことができます。
  - また、プロダクト環境のウェブサーバからコンテナを実行し、3000 番ポートを表示させることで、プロダクト環境に利用することもできます。
