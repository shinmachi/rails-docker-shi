# memo for M1 Mac 
* githubにプロジェクトを作成
  * 最小限のファイルから始める
* railsをローカルで立ち上げる
  * docker-compose run web rails new . --force --database=mysql
      * rm src/.gitignore
      * rm src/.gitattributes
      * rm -rf src/.git
  * docker-compose build 
  * docker-compose run web rails db:create
* herokuで正常動作させる
* circleciで、CIを動作させる
* circleciで、CDを動作させる

# 初期のGemfileの内容
```
source 'https://rubygems.org'

gem 'rails', '~> 6.1.0'
```



事前準備 
  githubに登録
  git config を設定
  herokuに登録
    クレカも登録する。herokuでは、mySQLをアドオンとして追加する。登録しないとmySQLがアドオンとして追加できない
  Hroku CLIをインストール

Herokuにログイン
  heroku login 
  heroku container:login
Herokuアプリを作成
  heroku create rails-docker-dik 

DBを追加・設定
  heroku addons:create cleardb:ignite -a rails-docker-dik
  heroku config -a rails-docker-dik
    mysql://b7474c89f53a7a:49c353b5@us-cdbr-east-04.cleardb.com/heroku_07695652adc3a51?reconnect=true    
    mysql://<user-name>:<password>@<host-name>/<database-name>
  heroku configに設定
    heroku config:add APP_DATABASE='heroku_07695652adc3a51' -a rails-docker-dik
    heroku config:add APP_DATABASE_USERNAME='b7474c89f53a7a' -a rails-docker-dik
    heroku config:add APP_DATABASE_PASSWORD='49c353b5' -a rails-docker-dik
    heroku config:add APP_DATABASE_HOST='us-cdbr-east-04.cleardb.com' -a rails-docker-dik
    heroku config -a rails-docker-dik
  
Dockerfileを本番環境用に修正
  start.shに本番環境特有の処理をさせる
  assets:precompileが本番で起動するようになる
    heroku config:add RAILS_SERVE_STATIC_FILES='true' -a rails-docker-dik 
  Change boot timeout
    https://tools.heroku.support/limits/boot_timeout
    => 120 secに変更 
  server.pidが残っていると、herokuでエラーになる
    rm src/tmp/pids/server.pid

Dockerイメージをビルド・リリース
  Dockerイメージをビルドして、コンテナレジストリにプッシュ
    heroku container:push web -a rails-docker-dik
  Dockerコンテナをheroku上にリリース
    heroku container:release web -a rails-docker-dik
  データベースのテーブルを更新したい時
    heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-dik
      Error: Exec format error
      => $ heroku ps -a rails-docker-dik 
      => web.1: crashed 2021/08/08 13:02:40 +0900 (~ 52s ago)
  ## エラー対応 ##
    マルチプラットフォームのイメージビルド
    $ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
    [+] Building 750.0s (12/12) FINISHED １２分かかった
    イメージにタグを付ける (イメージのコピー)
      docker tag d.shinmachi/rails-docker-dik registry.heroku.com/rails-docker-dik/web
    プッシュする
      docker push registry.heroku.com/rails-docker-dik/web
    リリースする
      heroku container:release web -a rails-docker-dik
    データベースのテーブルを更新
      heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-dik
    通った！！
  herokuのアプリを開く
    $ heroku open -a rails-docker-dik 
  ログをコンフィグに設定
    $ heroku config:add RAILS_LOG_STDOUT='true' -a rails-docker-dik
  ログを表示
    heroku logs -t -a rails-docker-dik
機能追加
  コントローラーを作る
    docker-compose exec web bundle exec rails g controller users
  ルーティングの設定 routes.rb 


  docker-compose down
  rm src/tmp/pids/server.pid
  docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
  docker tag d.shinmachi/rails-docker-dik registry.heroku.com/rails-docker-dik/web
  docker push registry.heroku.com/rails-docker-dik/web
  heroku container:release web -a rails-docker-dik
  heroku open -a rails-docker-dik 


Tips
dockerイメージのコピー（タグ名変更）
  docker tag [対象イメージ名:タグ] [変更後イメージ名:タグ]

M1の場合、Docker イメージをビルドする時は、プラットフォームの指定に気をつけないといけない。
  $ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
