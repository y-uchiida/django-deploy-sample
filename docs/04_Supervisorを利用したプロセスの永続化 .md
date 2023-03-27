# 04: Supervisor を利用したプロセスの永続化

## このドキュメントの内容

ここまでの設定では、SSH のターミナルを終了すると Django のプログラムが停止してしまいます。
Supervisor というプロセス監視・永続化ツールを利用して、Django アプリケーションの起動状態を維持できるようにします。

## 1. Supervisor のインストール

EC2 インスタンスに Supervisor をインストールします。

```bash
sudo apt-get install supervisor
```

## 2. Supervisor の設定ファイルの作成

今回は、Git リポジトリにプッシュしてある設定ファイルを利用します。  
[supervisor_config/django-deploy-sample.conf](/supervisor_config/django-deploy-sample.conf)  
このファイルが Supervisor から参照できるように、シンボリックリンクを貼ります。

```bash
sudo ln -s /home/ubuntu/django-deploy-sample/supervisor_config/django-deploy-sample.conf /etc/supervisor/conf.d/django-deploy-sample.conf
```

## 3. Supervidor の起動

以下のコマンドで、Supervisor を再起動します。

```bash
sudo systemctl supervisor restart
```

以下のコマンドで、Supervisor が uWSGI のプログラムを起動できているか確認します。

```bash
sudo supervisorctl status
```

以下の表に表示されれば、プログラムは起動しています。

```
django-deploy-sample             RUNNING   pid 626, uptime 2:26:13
```

また、EC2 インスタンスが起動した際に Supervisor も自動起動するように設定します。

```bash
sudo systemctl enable supervisor
```

## 4. 接続確認

以下の URL にアクセスして、動作を確認してみます。
`http://<EC2 インスタンスのパブリック IP>`  
Django のアプリケーションの画面がブラウザに表示されたら、正しく設定できています。

## 5. EC2 インスタンスの再起動と Django アプリの表示の確認

Django アプリケーションの自動起動と永続化が設定できたことを確認します。

EC2 インスタンスに接続している SSH コンソールをすべて終了し、EC2 インスタンスを停止します。
停止した後、EC2 インスタンスを起動しなおします。  
起動できたら、ブラウザから EC2 インスタンスのパブリック IP アドレスにアクセスします。

Django の画面が表示されれば、自動起動の設定が有効に動作しています。

※ Elactic IP を設定していない場合、EC2 インスタンスを起動しなおすと、パブリック IP アドレスが変更されるので、ご注意ください
