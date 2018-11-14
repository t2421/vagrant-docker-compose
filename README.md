# vagrant-docker-compose
dockerをwindowsおよびmacosでそのまま走らせると環境依存でうまくいかないことがあるらしいので、vagrantで仮想環境を作成して、その中でdockerを動作させることで環境依存を吸収する。　　

+ vagrant
+ dockerのインストール
+ docker-composeのインストール
+ laradockからtestappを使用できるようにする(別途管理)

## Vagrant設定(Vagrantfile)
`config.vm.network "public_network"`によってほかのデバイスからもプレビューできるように動的にipを割り当てる。  
`vagrant ssh`したあとに`ifconfig`でそれらしいipを発見する。
### 共有フォルダ
`config.vm.synced_folder "./", "/vagrant", owner: "vagrant", group: "vagrant"`  
Vagrantfileがある場所を仮想環境の /vagrant/と共有
### provisionについて
docker-composeまで使える環境を作りたかったので、`vagrant up`時に/provision/フォルダ内のシェルスクリプトを走らせる。  
これによりdockerのインストールおよびdocker-composeのインストールまでを行う。

## laradockの設定
apps/shell/instal-laradock.shに.envを生成するまでを記述しているので実行。  
.envのMYSQL_VERSIONはlatestにすると不都合があるので5.7とかにしておく。  
APP_CODE_PATH_HOSTについては、実際開発を進めたいフォルダに設定する。そうすると、workspaceにsshで入った時にそのフォルダが/var/www/になる。  
反映させるにはdocker-composeのstopとupが必要になる。  
testappフォルダを一応入れておく。  
.envはmysqlのところだけ修正必要
@see https://qiita.com/Dok1211/items/1041e857862eb18e69df  
なんか動かないなーと思ったら、testappフォルダ内でcomposer install をしていなかった。  
apacheのdocumentRootを変更するのは、laradockの.envファイル default.apache.confを修正するという記事もあったけどそれだと反映されなかった。

## mysqlコンテナにworkspaceコンテナからアクセスする
workspaceに配置されているlaravelアプリの.envファイルのmysql設定を以下のようにしていた。  
`DB_HOST=127.0.0.1`  
そうすると`php artisan migrate`を実行してもDBにつながらないというエラーが出た。  
いろいろ調べると、host名にはdockerのコンテナ名が必要だった。  
なので、`docker-compose ps`を使って、mysqlコンテナの名前を調べて、設定してあげたらできた。

## コンテナ起動
`docker-compose up -d workspace apache2 mysql`

## dockerコマンドにいちいちsudoをつけたくない
https://qiita.com/DQNEO/items/da5df074c48b012152ee

## コンテナに入る
`docker exec -i -t コンテナ名(docker-compose psで見れる) bash`  
composerなどを利用したい場合はworkspaceコンテナに入らないと実行できない。

## JWTを使って認証を行う
https://jwt-auth.readthedocs.io/en/docs/  
Laravel Passportだとうまくいかなかったのでこっちを使ったらうまくいった。

## vagrant内でnpm install ができない
windowsから見えている共有フォルダではパス長の問題とシンボリックリンクが貼れない問題からエラーが出る。  
なので、一旦windowsから見えないフォルダで`npm install --no-bin-links`でインストールしてフォルダを移動する(ただし時間がすごいかかる)ということを行った。  
また、node-sassのインストールでエラーが出ていたが、
```
npm config set user 0
npm config set unsafe-perm true
```
を実行することでなぜか成功した。