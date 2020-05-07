# php-apache-mssql 接続サンプル

## 概要

- `php-xdebug`で利用したコンテナを改修した`php-apache`コンテナを、`mssql`コンテナのデータベースに接続するデモです。
  - 差分は以下の通りです。
    - `Dockefile`
      - mssql ドライバを追加
    - `src/index.php`
      - 下記参考ドキュメントの接続確認ページを利用しました。
    - `docker-compose.yml`
      - `mssql`コンテナの定義を追加
- アクセス可能ポートを変更し、[http://localhost:8083](http://localhost:8083)としています。
- これをベースにして、コンテナの mssql ではなく既存データベースを利用したい場合、
  - 接続文字列を変更してください。
  - `docker-compose.yml`内の`mssql`コンテナ定義を削除してください。
- 基本的な操作方法は、ベースである[docker_xdebug](../docker_xdebug/)のチュートリアルを参照してください。

## 備考

- `php-apache`コンテナでの設定手順は Microsoft 公式ドキュメントを参照しました。
  - https://docs.microsoft.com/ja-jp/sql/connect/php/installation-tutorial-linux-mac?view=sql-server-ver15
- `mssql`コンテナ構築については、`dockerhub`内のドキュメントを参照しました。
  - https://hub.docker.com/_/microsoft-mssql-server