# memo for M1 Mac 
* githubにプロジェクトを作成
  * 最小限のファイルから始める
* railsをローカルで立ち上げる
  * docker-compose run web rails new . --force --database=mysql
      * rm src/.gitignore
      * rm src/.gitattributes
      * rm -rf src/.git
      * まちがえてmysql_dataも管理してしまったので、管理から外す
      * git rm -r --cached src/db/mysql_data
  * docker-compose build 
  * modified database.yml  
  * docker-compose run web rails db:create
  * docker-compose up 
* Hello world!を表示させる
  * docker-compose exec web bundle exec rails g controller users
  * ルーティングの設定 routes.rb 
* herokuで正常動作させる
  * Herokuにログイン
    * heroku login 
    * heroku container:login
  * Herokuアプリを作成
    * heroku create rails-docker-shin
    * プロジェクト名が既に使われていてため、変更した
      * git remote -v で確認
      * git remote set-url origin <new-url>
    * DBを追加する
      * heroku addons:create cleardb:ignite -a rails-docker-shin
      * heroku config -a rails-docker-dik
        * mysql://bb17eb18cdaaf3:800be9bc@us-cdbr-east-04.cleardb.com/heroku_bc3d048e4861bc2?reconnect=true
        * mysql://<user-name>:<password>@<host-name>/<database-name>
      * heroku configに設定
        * heroku config:add APP_DATABASE='heroku_bc3d048e4861bc2' -a rails-docker-shin
        * heroku config:add APP_DATABASE_USERNAME='bb17eb18cdaaf3' -a rails-docker-shin
        * heroku config:add APP_DATABASE_PASSWORD='800be9bc' -a rails-docker-shin
        * heroku config:add APP_DATABASE_HOST='us-cdbr-east-04.cleardb.com' -a rails-docker-shin
        * heroku config -a rails-docker-shin
    * Dockerfileを本番環境用に修正
      * Dockerfileに環境変数とスクリプト実行コマンドを追加する
      * start.shに本番環境特有の処理をさせる
      * assets:precompileが本番で起動するようになる
        * heroku config:add RAILS_SERVE_STATIC_FILES='true' -a rails-docker-shin
      * Change boot timeout => 120 secに変更
        * https://tools.heroku.support/limits/boot_timeout
      * server.pidが残っていると、herokuでエラーになる
        * rm src/tmp/pids/server.pid
    * Dockerイメージをビルド・リリース
      * Dockerイメージをビルドして、コンテナレジストリにプッシュ
        * heroku container:push web -a rails-docker-shin
      * Dockerコンテナをheroku上にリリース
        * heroku container:release web -a rails-docker-shin
      * herokuのアプリを開く
        * heroku open -a rails-docker-shin
        * => Error: Exec format error
    * エラーへの対応
      * マルチプラットフォームのイメージビルド
        * docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-shin:latest
      * イメージにタグを付ける (イメージのコピー)
        * docker tag d.shinmachi/rails-docker-shin registry.heroku.com/rails-docker-shin/web
      * プッシュする
        * docker push registry.heroku.com/rails-docker-shin/web
      * リリースする
        * heroku container:release web -a rails-docker-shin
      * herokuのアプリを開く
        * heroku open -a rails-docker-shin
        * => Hello world!
* circleciで、CIを動作させる
  * テストコードを作る
    * docker-compose exec web bundle exec rake test
  * .circleci/config.ymlを作成
    * circleci-cliをインストール => config.ymlのバリデーションのため
      * brew install circleci
      * curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash
    * config.ymlのバリデーション
      * circleci config validate
    * 
  ## 環境変数を設定
* circleciで、CDを動作させる

## テストを実行した時の挙動
原因はよくわかっていない

```
$ docker-compose exec web bundle exec rake test
Run options: --seed 63488

# Running:

/app/db/schema.rb doesn't exist yet. Run `bin/rails db:migrate` to create it, then try again. If you do not intend to use a database, you should instead alter /app/config/application.rb to limit the frameworks that will be loaded.
```

# Tips
### dockerイメージのコピー（タグ名変更）
`docker tag [対象イメージ名:タグ] [変更後イメージ名:タグ]`

### M1の場合、Docker イメージをビルドする時は、プラットフォームの指定に気をつけないといけない。
```
$ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
```


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
  $ heroku addons:create cleardb:ignite -a rails-docker-shin
  heroku config -a rails-docker-dik
    mysql://bb17eb18cdaaf3:800be9bc@us-cdbr-east-04.cleardb.com/heroku_bc3d048e4861bc2?reconnect=true
    mysql://<user-name>:<password>@<host-name>/<database-name>
  heroku configに設定
    heroku config:add APP_DATABASE='heroku_bc3d048e4861bc2' -a rails-docker-shin
    heroku config:add APP_DATABASE_USERNAME='bb17eb18cdaaf3' -a rails-docker-shin
    heroku config:add APP_DATABASE_PASSWORD='800be9bc' -a rails-docker-shin
    heroku config:add APP_DATABASE_HOST='us-cdbr-east-04.cleardb.com' -a rails-docker-shin
    heroku config -a rails-docker-shin
  
* Dockerfileを本番環境用に修正
  * Dockerfileに環境変数とスクリプト実行コマンドを追加する
  * start.shに本番環境特有の処理をさせる
  * assets:precompileが本番で起動するようになる
    * heroku config:add RAILS_SERVE_STATIC_FILES='true' -a rails-docker-shin
  * Change boot timeout => 120 secに変更
    * https://tools.heroku.support/limits/boot_timeout
  * server.pidが残っていると、herokuでエラーになる
    * rm src/tmp/pids/server.pid

* Dockerイメージをビルド・リリース
  * Dockerイメージをビルドして、コンテナレジストリにプッシュ
    * heroku container:push web -a rails-docker-shin
  * Dockerコンテナをheroku上にリリース
    * heroku container:release web -a rails-docker-shin
  * herokuのアプリを開く
    * heroku open -a rails-docker-shin
    * => Error: Exec format error

* エラーへの対応
  * マルチプラットフォームのイメージビルド
    * docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-shin:latest
  * イメージにタグを付ける (イメージのコピー)
    * docker tag d.shinmachi/rails-docker-shin registry.heroku.com/rails-docker-shin/web
  * プッシュする
    * docker push registry.heroku.com/rails-docker-shin/web
  * リリースする
    * heroku container:release web -a rails-docker-shin
  * herokuのアプリを開く
    * heroku open -a rails-docker-shin
    * => Hello world!


  データベースのテーブルを更新したい時
    heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-shin
      Error: Exec format error
      => $ heroku ps -a rails-docker- shin
      => web.1: crashed 2021/08/08 13:02:40 +0900 (~ 52s ago)
  ## エラー対応 ##
    マルチプラットフォームのイメージビルド
    $ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-shin:latest
    [+] Building 750.0s (12/12) FINISHED １２分かかった
    イメージにタグを付ける (イメージのコピー)
      docker tag d.shinmachi/rails-docker- shinregistry.heroku.com/rails-docker-/shinweb
    プッシュする
      docker push registry.heroku.com/rails-docker-/shinweb
    リリースする
      heroku container:release web -a rails-docker-shin
    データベースのテーブルを更新
      heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-shin
    通った！！
  herokuのアプリを開く
    $ heroku open -a rails-docker- shin
  ログをコンフィグに設定
    $ heroku config:add RAILS_LOG_STDOUT='true' -a rails-docker-shin
  ログを表示
    heroku logs -t -a rails-docker-shin
機能追加
  コントローラーを作る
    docker-compose exec web bundle exec rails g controller users
  ルーティングの設定 routes.rb 


  docker-compose down
  rm src/tmp/pids/server.pid
  docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-:shinlatest
  docker tag d.shinmachi/rails-docker- shinregistry.heroku.com/rails-docker-/shinweb
  docker push registry.heroku.com/rails-docker-/shinweb
  heroku container:release web -a rails-docker-shin
  heroku open -a rails-docker- shin

