# 01: ローカルの開発環境準備と、Django プロジェクトの初期作成

## このドキュメントの内容

チーム開発を行う場合の、Django アプリケーションの初期セットアップの手順例を示します。  
Poetry を利用してパッケージ管理を行います。  
また、セキュリティ保持および環境別の設定切替の効率化のため、django-environ の設定も行います。

## 1. Poetry をインストールする

Python 用のパッケージ管理ツール Poetry をインストールします。  
Poetry の詳細は、以下の付属資料をご覧ください。  
[Python_Django の環境構築（チーム開発・大規模開発向け）.pdf](/docs/Python_Django%E3%81%AE%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89%EF%BC%88%E3%83%81%E3%83%BC%E3%83%A0%E9%96%8B%E7%99%BA%E3%83%BB%E5%A4%A7%E8%A6%8F%E6%A8%A1%E9%96%8B%E7%99%BA%E5%90%91%E3%81%91%EF%BC%89.pdf)

インストールのため、以下のコマンドを実行します。

- MacOS, WSL など Unix ベース 環境を利用する場合

  ```bash
  curl -sSL https://install.python-poetry.org | python3 -
  ```

- Powershell を利用する場合

  ```powershell
  (Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -
  ```

## 2. poetry で プロジェクトを作成する

パッケージバージョンの共通化のため、Poetry を利用してプロジェクトを管理します。

- poetry プロジェクトを作成します

  ```bash
  poetry new django-deploy-sample
  ```

- 作成したプロジェクトにカレントディレクトリを移動します

  ```bash
  cd django-deploy-sample
  ```

- poetry コマンドを利用して Django パッケージをインストールします

  ```bash
  poetry add django==3.2
  ```

- 環境による設定差分を管理するため、poetry コマンドを利用して django-environ をインストールします
  ```bash
  poetry add django-environ
  ```

## 3. Django アプリケーションを作成する

- poetry コマンドを利用して、Django プロジェクトを作成します

  ```bash
  poetry run django-admin startproject config .
  ```

- アプリケーションの起動を確認します
  ```bash
  poetry run python manage.py runserver
  ```

## 4. .gitignore を用意する

機密情報が GitHub リポジトリに保持されることがないように、.gitignore を作成します。  
Django ではデフォルトの.gitignore 設定がないため、手動で追加する必要があります。  
以下の URL から、Django プロジェクト向けの.gitignore の設定例を作成できます。  
https://www.toptal.com/developers/gitignore

## 5. 接続情報を.env に記載する

.env ファイルを作成し、GitHub リモートリポジトリ に公開すべきでない項目を設定します。  
.env ファイルは.gitignore の設定によって無視され、リモートリポジトリに公開されません。  
環境間の設定をスムーズに行えるように、ひな形は別のファイル名にしてプッシュします。  
設定値のひな形ファイルは、`.env.example` などの名称にしておきます。

## 6. settings.py の修正

`.env` ファイルの設定を読み込むように、`config/settings.py` の設定を修正します。

```python
# environ パッケージを読み込み
import environ

# 以下のように記述して、 .env ファイルの内容を参照できるオブジェクトを用意する
env = environ.Env()
env.read_env(".env")

# ... 中略 ...

# env() を利用して設定値を差し込む
SECRET_KEY = env("SECRET_KEY")
```

## 7. リモートリポジトリへのコードのプッシュ

設定ができたら、Github にコードを push します。

```bash
git init .
git remote add origin ＜リモートリポジトリのアドレス＞
git add .
git commit -m "first commit"
git branch -m master main
git push origin main
```
