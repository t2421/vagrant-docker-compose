# vagrant-docker-compose
dockerをwindowsおよびmacosでそのまま走らせると環境依存でうまくいかないことがあるらしいので、vagrantで仮想環境を作成して、その中でdockerを動作させることで環境依存を吸収する。　　

+ vagrant
+ dockerのインストール
+ docker-composeのインストール

## Vagrant設定(Vagrantfile)
`config.vm.network "public_network"`によってほかのデバイスからもプレビューできるように動的にipを割り当てる。  
`vagrant ssh`したあとに`ifconfig`でそれらしいipを発見する。
### 共有フォルダ
`config.vm.synced_folder "./", "/vagrant", owner: "vagrant", group: "vagrant"`  
Vagrantfileがある場所を仮想環境の /vagrant/と共有
### provisionについて
docker-composeまで使える環境を作りたかったので、`vagrant up`時に/provision/フォルダ内のシェルスクリプトを走らせる。  
これによりdockerのインストールおよびdocker-composeのインストールまでを行う。


## dockerコマンドにいちいちsudoをつけたくない
https://qiita.com/DQNEO/items/da5df074c48b012152ee

## コンテナに入る
`docker exec -i -t コンテナ名(docker-compose psで見れる) bash`  
composerなどを利用したい場合はworkspaceコンテナに入らないと実行できない。

## vagrant内でnpm install ができない
windowsから見えている共有フォルダではパス長の問題とシンボリックリンクが貼れない問題からエラーが出る。  
なので、一旦windowsから見えないフォルダで`npm install --no-bin-links`でインストールしてフォルダを移動する(ただし時間がすごいかかる)ということを行った。  
また、node-sassのインストールでエラーが出ていたが、
```
npm config set user 0
npm config set unsafe-perm true
```
を実行することでなぜか成功した。
