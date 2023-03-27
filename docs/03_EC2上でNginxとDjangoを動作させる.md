# 03: EC2 上で Nginx と Django を動作させる

## このドキュメントの内容

EC2 インスタンス上で Nginx を起動させ、ブラウザからのアクセスを受付けることができるようにします。  
さらに、Nginx から uWSGI に処理を受け渡し、Django の実行結果をブラウザにレスポンスできるようにします。

## 1. Nginx の設定ファイルを作成

今回は、Git リポジトリにプッシュしてある設定ファイルを利用します。  
[nginx_config/django-deploy-sample.conf](/nginx_config/django-deploy-sample.conf)  
Nginx がアクセスを受付けた際に、uWSGI アプリケーションを呼び出す設定を記述しています。  
このファイルが Nginx から参照できるように、シンボリックリンクを貼ります。

```bash
sudo ln -s /home/ubuntu/django-deploy-sample/nginx_config/django-deploy-sample.conf /etc/nginx/sites-enabled/
```

さらに、初期設定の Nginx の設定ファイルと競合しないよう、シンボリックリンクを切って初期設定ファイルを無効化します。

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Nginx の設定ファイルの設定確認コマンドでエラーが出ないかを確認します。

```bash
sudo nginx -t
```

また、EC2 インスタンスが起動した際に Nginx も自動起動するように設定しておきます。

```bash
sudo systemctl enable nginx
```

なお、このリポジトリではあらかじめ uWSGI 用の設定ファイルを用意していますが、  
初期状態ではファイルが容易されていません。  
Nginx のディレクトリに保存されているので、以下のコマンドでコピーします。  
（GitHub から clone してきた場合は、以下のコマンドの実行は不要です）

```bash
cp /etc/nginx/uwsgi_params /home/ubuntu/django-deploy-sample/
```

## 2. Nginx の再起動と接続確認

設定の変更を反映させるため、Nginx を再起動します。

```bash
sudo systemctl restart nginx
```

この時点で、Nginx は ポート 80 の Listen を開始しています。  
以下の URL にアクセスして、動作を確認してみます。
`http://<EC2 インスタンスのパブリック IP>`  
Django のアプリケーションを起動していないため、502 Bad Gateway が表示されます。

## 3. uWSGI の起動と接続確認

uWSGI を利用して、Django アプリケーションを起動します。
以下のコマンドを実行します。

```bash
poetry run uwsgi --master --socket=:8001 --plugin=python3 --chdir=/home/ubuntu/django-deploy-sample --module=config.wsgi
```

以下の URL にアクセスして、動作を確認してみます。
`http://<EC2 インスタンスのパブリック IP>`  
Django のアプリケーションの画面がブラウザに表示されたら、正しく設定できています。  
Ctrl + C キーを押して、uWSGI を終了すると、ターミナルが操作できるようになります。
