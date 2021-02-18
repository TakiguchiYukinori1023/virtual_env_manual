
# 環境構築手順書


## バージョン一覧
|名前|バージョン|
|:-----------|:-----------|
|PHP|7.3|
|Nginx|1.19|
|MySQL|5.7|
|Laravel|6.2|
|OS|CentOS7|


## 手順

(以下はホストOS内での作業)

### Vagrant boxのダウンロード
1. 任意のディレクトリで```vagrant box add centos/7```を実行
1. virtualboxを選択

### Vagrantの作業用ディレクトリを用意
1. 自分の作業用のディレクトリで```mkdir vagrant_task```を実行
1. vagrant_taskフォルダに移動後、```vagrant init centos/7```を実行
1. vagrantfileの```config.vm.network "forwarded_port", guest: 80, host: 8080```と ```config.vm.network "private_network", ip: "192.168.33.10"```の2箇所をコメントアウトしipを192.168.33.19に変更する
1. ```config.vm.synced_folder "../data", "/vagrant_data"```を```config.vm.synced_folder "./", "/vagrant", type:"virtualbox"```に編集

### Vagrantプラグインのインストール
1. ```vagrant plugin install vagrant-vbguest```を実行
1. ```vagrant plugin install vagrant-global-status```を実行
1. ```vagrant plugin install sahara```を実行

### Vagrantを使用してゲストOSを起動とログイン
1. ```vagrant up```を実行
1. vagrant_taskフォルダ下で```vagrant ssh```を実行

(以下はゲストOS内での作業)

### パッケージをインストール
1. ```sudo yum -y groupinstall "development tools"```を実行

### PHP7.3をインストール
1. ```sudo yum -y install epel-release wget```を実行
1. ```sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm```を実行
1. ```sudo rpm -Uvh remi-release-7.rpm```を実行
1. ```sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip```を実行
1. ```php -v```を実行してバージョン7.3がインストール出来ているか確認

### Composerのインストール
1. ```php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"```を実行
1. ```php composer-setup.php```を実行
1. ```php -r "unlink('composer-setup.php');"```を実行
1. ```sudo mv composer.phar /usr/local/bin/composer```を実行
1. ```composer -v```を実行してバージョンが表示されるか確認

### Laravel6.2をインストール
1. ```composer create-project laravel/laravel --prefer-dist laravel_task 6.2```を実行
1. laravel_taskフォルダに移動後、```php artisan --version```を実行してバージョン6.2がインストール出来ているか確認
1. ```mv /home/vagrant/laravel_task /vagrant```を実行しlaravel_taskディレクトリを/vagrantディレクトリ下に移動させる

### Nginxのインストール
1. viエディタを使用して```sudo vi /etc/yum.repos.d/nginx.repo```ファイルを作成
1. 作成したファイルに 
```[nginx] 
name=nginx repo 
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/ 
gpgcheck=0 
enabled=1
``` 
を書き込み保存
1. ```sudo yum install -y nginx```を実行してnginxをインストール
1. ```nginx -v```を実行してバージョンを確認
1. ```sudo systemctl start nginx```を実行してブラウザにて http://192.168.33.19 と入力しNginxのwelcomeページが表示されるか確認

### Nginxとphp-fpmの設定ファイルの編集
1. ```sudo vi /etc/nginx/conf.d/default.conf```を実行しNginxの設定ファイルを編集
1. 設定ファイルの以下の箇所を編集
``` server {
listen       80;
server_name  192.168.33.10; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
# ApacheのDocumentRootにあたります
root /vagrant/laravel_app/public; # 追記
index  index.html index.htm index.php; # 追記
#charset koi8-r;
#access_log  /var/log/nginx/host.access.log  main;
location / {
    #root   /usr/share/nginx/html; # コメントアウト
    #index  index.html index.htm;  # コメントアウト
    try_files $uri $uri/ /index.php$is_args$args;  # 追記
}
# 省略
# 該当箇所のコメントを解除し、必要な箇所には変更を加える
# 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。
location ~ \.php$ {
#   root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
    include        fastcgi_params;
}
# 省略
```
1. ```sudo vi /etc/php-fpm.d/www.conf```を実行しphp-fpmの設定ファイルを編集
1. 設定ファイルの以下の箇所を編集
```
;24行目近辺
user = apache
# ↓ 以下に編集
user = vagrant
group = apache
# ↓ 以下に編集
group = vagrant
```
1. ```sudo systemctl restart nginx```を実行しNginxを再起動
1. ```sudo systemctl start php-fpm```を実行しphp-fpmモジュールを起動

###　権限の設定
1. /vagrant/laravel_appディレクトリに移動して、```sudo chmod -R 777 storage```を実行
1. ```sudo chmod 777 /var```を実行
1. ```sudo chown vagrant:vagrant /var```を実行

### Mysqlのインストール
1. ```sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm```を実行しバージョン5.7を指定
1. ```sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm```を実行しバージョン5.7を指定
1. ```sudo yum install -y mysql-community-server```を実行
1. ```mysql --version```を実行しバージョンが5.7になっているか確認
1. ```sudo systemctl start mysqld```を実行しmysqlを起動
1. ```sudo vi /etc/my.cnf```を実行し```validate-password=OFF```を実行しパスワード設定の際の条件を無くす
1. ```sudo systemctl restart mysqld```を実行しmysqlを再起動
1. ```sudo cat /var/log/mysqld.log | grep 'temporary password'```を実行し表示される  
``` A temporary password is generated for root@localhost:```以降のランダムな文字列をコピー
1. ```mysql -u root -p```を実行後表示されるパスワードにコピーしたランダムな文字列をペーストしログイン
1. ```set password = ""```を実行しパスワードを設定 

### データベースの作成とLaravelプロジェクトの設定ファイル編集
1. ```create database laravel_task;```を実行してデータベースを作成
1. laravel_taskディレクトリ下の```.env```ファイルの内容を以下に変更 
```
DB_DATABASE=laravel
# ↓ 以下に編集
DB_DATABASE=laravel_task
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```
1. laravel_taskディレクトリに移動後、```php artisan migrate```を実行
1. ブラウザ上でユーザー登録出来るか確認 

### Laravelにログイン機能を実装
1. laravel_taskディレクトリ下で```composer require laravel/ui "^1.0" --dev```を実行
1. ```php artisan ui vue --auth```を実行
1. ブラウザでユーザー登録とログインが出来るか確認 

## 所感
今までのレッスンではGUIの操作が基本だったので、目で見えて分かりやすかったのですがサーバーレッスン  
に入ってからは全ての操作をターミナルのコマンドで行うので、ディレクトリの配置などをlsコマンドで  
確認しながら行ったりviエディタを使用してファイルの編集を行ったりするのは慣れるのが大変でした。  

レッスンをこなしていくうちにディレクトリの配置や「この作業をするにはどのディレクトリに移動して  
何のコマンドを打てば良いか」を徐々に頭の中で考えられるようになりました。
  
レッスンの注意書きでもありましたが、「今自分がホストOSとゲストOSのどちらで作業をしているか」、  
「インストールするパッケージのバージョンは間違っていないか」、  
「sudoを付けて実行するコマンドやディレクトリ削除するコマンドは慎重に実行する」などの操作に気をつけて、  
sahara等のプラグインを上手く利用しながら今後のターミナル操作、仮想環境構築に取り組みたいと思いました。

## 参考サイト
- 【Markdown】表を作る方法とは？文字の表示位置指定方法も解説! https://www.sejuku.net/blog/77323
- Excelのお作法から解放されたい、MarkDownで作業手順書を作成 https://qiita.com/haruto167/items/ecf5b0434e1a544edc4f
- Vagrantで起動しているVMを一覧する https://qiita.com/ringo/items/e30761b89fb6c9a1c45d
- [Vagrant]saharaプラグインで仮想OS状態を管理する https://dev.classmethod.jp/articles/vagrant-sahar/
- Laravel 6.x 認証 https://readouble.com/laravel/6.x/ja/authentication.html
- 更新！！Laravel6/7 (laravel/ui)でのLogin機能の実装方法〜MyMemo　https://qiita.com/iaoiui/items/595ecddb9e7064279fd0
