# 02: AWS の環境構築

## このドキュメントの内容

AWS で EC2 インスタンスと RDS インスタンスを作成し、設定を行います。  
EC2 インスタンス上に GitHub リポジトリのソースコードを clone し、開発用サーバーを起動するところまで解説します。

## 1. RDS インスタンスを作成

AWS 上に、RDS インスタンスを作成します。  
エンジンスタイプは、PostgresSQL を選択します。  
![SQLエンジンタイプの選択](/docs/images/02-01_setting-rds-instance.png "SQLエンジンタイプの選択")

## 2. EC2 インスタンスを作成

AWS 上に、Ubuntu のインスタンスを作成します。  
![EC2インスタンスタイプの選択](/docs/images/02-02_setting-ec2-instance-1.png "EC2インスタンスタイプの選択")

接続用の SSH 鍵をダウンロードして、控えておきます。  
![SSH鍵の生成](/docs/images/02-03_setting-ec2-instance-2.png "SSH鍵の生成")

RDS インスタンスに接続できるように、セキュリティグループを追加する必要があります。  
詳細は、以下の AWS のドキュメントを参照してください。

- [Connect your EC2 instance to an AWS resource](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-instance-to-resources.html)
- [Amazon EC2 が EC2 インスタンスと RDS データベース間の自動接続設定ソリューションの提供を開始](https://aws.amazon.com/jp/about-aws/whats-new/2022/10/amazon-ec2-automated-connection-solution-ec2-instance-rds-database/)

## 3. インバウンドルールの追加

EC2 インスタンスが外部からのアクセスに対してレスポンスできるように、ポートを開放する必要があります。  
セキュリティグループの設定から、インバウンドルールを追加してください。  
![SSH FS の設定](/docs/images/02-04_setting-inbound-rules.png "SSH FS の設定")  
上記の設定例では、80, 8000, 8001 を設定していますが、ポート 80 だけ開放しておけば、Nginx からの接続テストを行うことができます。  
ポート 8000 および 8001 は、Django の開発サーバーに直接 HTTP アクセスして動作確認する場合に必要となります。  
設定方法の詳細は、以下の AWS のドキュメントを参照してください。

- [セキュリティグループへのルールの追加](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/working-with-security-groups.html#adding-security-group-rule)

## 4. EC2 インスタンスに SSH 接続する

VSCode を開いて、SSH FS にインスタンスの接続情報を設定します。  
![SSH FS の設定](/docs/images/02-05_ssh-setting-on-vscode.png "SSH FS の設定")

設定後、SSH FS でインスタンスに SSH 接続します。

## 5. EC2 インスタンスに、Django の実行に必要なアプリケーションをインストールする

EC2 インスタンスに SSH で接続したら、以下のコマンドを実行して Django を動作させる環境をセットアップします。

- Ubuntu のパッケージ一覧更新

  ```bash
  sudo apt-get update
  ```

- pip インストール

  ```bash
  sudo apt install python3-pip
  ```

- Nginx インストール

  ```bash
  sudo apt-get install nginx
  ```

- Postgre SQL サーバーに接続するためのコマンドをインストール

  ```bash
  sudo apt install postgresql-client
  ```

- poetry インストール

  ```bash
  curl -sSL https://install.python-poetry.org | python3 -
  ```

- poetry の実行ファイルにパスを通す

  ```bash
  echo 'PATH="/home/ubuntu/.local/bin:$PATH"' >> ~/.bashrc
  source ~/.bashrc
  ```

- bashrc の更新をターミナルに反映

  ```bash
  source ~/.bashrc
  ```

- poetry の起動確認
  ```bash
  poetry -v
  ```

## 6. GitHub からソースコードを取得

以下のようにコマンドを実行します。  
`git clone <GiHub リモートリポジトリのアドレス>`

下記は、このリポジトリを clone する場合の例です。

```bash
git clone https://github.com/y-uchiida/django-deploy-sample.git
```

## 7. 取得したソースコードで Django を動作できるようにセットアップ

GitHub から clone しただけでは、プロジェクトは動作しません。  
アプリケーションとして起動できるように、設定をします。

### poetry から、python パッケージのインストール

clone 直後は、Django などのパッケージのソースコードがインストールされていません。  
以下のコマンドで、Python パッケージをインストールします。

```bash
poetry install
```

### RDS にデータベースを作成する

RDS インスタンスに、Django が利用するデータベースを作成します。  
以下のコマンドを実行します。

- Postgres SQL サーバーに接続
  ```bash
  psql -h 【RDS サーバーのエンドポイント】 -p 5432 -U postgres -d postgres
  ```
- Django 用のデータベースを作成

  ```psql
  create database django_db;
  ```

- Postgres SQL サーバーと接続を終了
  ```psql
  exit;
  ```

### env ファイルの作成

リモートリポジトリには`.env` は公開され内容に設定してあるため、ファイルが存在していません。  
`env.example` をコピーして`.env` を作り、その内容を編集します。  
データベースの設定情報などは、RDS の設定画面から取得してください。  
RDS の画面で、作成したデータベース名をクリックすると詳細画面が開きます。  
ここから接続情報を確認してください。

### Django アプリケーションを起動する

ここまでの設定が適切に行えているか、実際に Django アプリケーションを起動して動作を確認してみます。

```
poetry run python manage.py runserver
```

エラーメッセージが出ることなく、 `127.0.0.1:8000` でアプリケーションが起動していれば、設定は完了です。

また、`http://<EC2 インスタンスのパブリックIPアドレス>:8000` で、Django アプリケーションの画面が表示されることを確認できます。  
なお、ブラウザからの接続するためには、8000 ポートの開放が必要になります。
