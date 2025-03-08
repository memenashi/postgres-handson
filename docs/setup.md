# 環境構築: DockerでPostgreSQLとpgAdminを起動

まずはPostgreSQLを操作するための環境を用意します。Dockerとdocker-composeを使って、PostgreSQLデータベース本体とGUI管理ツールであるpgAdminをセットアップします。

**手順:**

1. **Dockerのインストール**: まだインストールしていない場合は、公式サイトからDocker Desktop（Windows版）をインストールしてください。インストール後、Docker Desktopを起動しておきます。

2. **Docker Composeファイルの作成**: 任意の作業フォルダを作成し、その中に`docker-compose.yml`というファイルを新規作成します。以下の内容をファイルに記述してください（PostgreSQLとpgAdminの2つのサービスを定義します）。

    ```yaml
    version: '3'
    services:
      db:
        image: postgres:15  # PostgreSQLの公式イメージ（バージョン15を使用）
        environment:
          POSTGRES_PASSWORD: mysecretpassword  # PostgreSQLのパスワードを設定（任意の文字列に変更可）
        ports:
          - "5432:5432"  # ホストの5432番ポートをコンテナの5432番ポートに割り当て
      pgadmin:
        image: dpage/pgadmin4  # pgAdmin4の公式イメージ
        environment:
          PGADMIN_DEFAULT_EMAIL: admin@example.com  # pgAdminログイン用メールアドレス（任意に設定可）
          PGADMIN_DEFAULT_PASSWORD: admin           # pgAdminログイン用パスワード（任意に設定可）
        ports:
          - "5050:80"  # ホストの5050番ポートをコンテナの80番ポートに割り当て（pgAdminはコンテナ内で80番ポートで動作）
    ```

    上記では、PostgreSQLコンテナ（サービス名: `db`）とpgAdminコンテナ（サービス名: `pgadmin`）を定義しています。PostgreSQLにはパスワード（`POSTGRES_PASSWORD`）を設定し、pgAdminには初期ログイン情報（メールアドレスとパスワード）を環境変数で渡しています。ポートの設定により、ローカルPCのポート5432でPostgreSQLに、ポート5050でpgAdminにアクセスできるようになります。

3. **コンテナの起動**: コマンドプロンプトもしくはPowerShellで、`docker-compose.yml`を置いたディレクトリに移動し、次のコマンドを実行します。

    ```bash
    docker-compose up -d
    ```

    このコマンドで、定義した2つのコンテナをバックグラウンド（デタッチドモード）で起動します。初回はイメージのダウンロードが行われるため少し時間がかかりますが、メッセージにエラーが出なければOKです。

4. **pgAdminへのアクセス**: ブラウザを開き、アドレスバーに `http://localhost:5050` と入力してpgAdminにアクセスします。pgAdminのログイン画面が表示されたら、先ほど`PGADMIN_DEFAULT_EMAIL`と`PGADMIN_DEFAULT_PASSWORD`で設定した資格情報（例ではメール`admin@example.com`、パスワード`admin`）を入力してログインしてください。

5. **PostgreSQLサーバーの登録**: pgAdminにログインしただけでは、まだデータベースに接続されていません。pgAdmin上でPostgreSQLサーバーを登録しましょう。

    - 左側のサーバー一覧ペインで「**Servers**」を右クリックし、「**Create**」->「**Server...**」を選択します。  
    - 「General」タブで任意の名前をつけます（例: `Local PostgreSQL`）。  
    - 「Connection」タブで、以下の情報を入力します。  
      - **Host name/address**: `db`  
      - **Port**: `5432`  
      - **Username**: `postgres`  
      - **Password**: `mysecretpassword` （docker-compose.ymlで設定したパスワード）  
      - **Save password?**（パスワードを保存するか）: チェックしておくと次回から入力不要です。  
    - 入力ができたら「Save」をクリックします。

    サーバーを保存すると、pgAdmin左ペインの「Servers」配下に今登録したサーバー名が表示され、その中にデータベース一覧（初期状態では`postgres`というデータベースなど）が見えるはずです。これでpgAdminからPostgreSQLコンテナ内のデータベースに接続できました。

    > ※ 補足: PostgreSQLコンテナには、デフォルトで`postgres`という名前のデータベースと、`postgres`ユーザー（管理者）が存在しています。これらは先ほど設定したパスワードで保護されています。チュートリアルでは特に指示がない限り、このデフォルトの`postgres`データベースを使って操作を行います。

6. **接続確認**: 実際にSQLを実行して接続を確認してみましょう。pgAdminで登録したサーバー配下のデータベース（例: `postgres`）を右クリックし、「**Query Tool**（クエリツール）」を開きます。新しく開いたクエリ編集画面で、次のSQL文を入力して実行ボタン（三角の再生マーク）をクリックしてください。

    ```sql
    SELECT version();
    ```

    これはPostgreSQLのバージョンを取得するSQLです。下部の結果パネルにPostgreSQLのバージョン情報が表示されれば、環境構築と接続は成功です。

以上で、学習を進める準備が整いました。次章からSQLの基本を学んでいきましょう。
