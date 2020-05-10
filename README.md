# PHP ユーザ向け Docker チュートリアル

- 原則として上から順に内容が詳細になりますが、`PHP` 環境に関心がない場合、[2. `Docker`コンテナを作成してみる](2. `Docker`コンテナを作成してみる)からチュートリアルを開始します。

## 1. 基本的な利用を試してみる
- [Docker,VSCode で XDebug を利用する](./docker_xdebug)
  - `Docker for Windows`の導入、`vscode`を利用したウェブサーバコンテナの`PHP`デバッグ
- [docker_xdebug コンテナを元に mssql に接続する](./php-mssql)
  - `/docer_xdebug`チュートリアルのコンテナに`mssql`接続機能を追加する
  
## 2. `Docker`コンテナを作成してみる
- [コンテナ作成の手順を確認する](./docker_workflow)
  - コンテナ作成から実行まで、一連の流れを解説
- [実用例から見る Dockerfile](./dockerfile_in_reality)
  - 東京都COVIDプロジェクトの`Dockerfile`の内容を解説
