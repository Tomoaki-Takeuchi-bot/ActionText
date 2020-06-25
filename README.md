
```
docker-compose exec web ./bin/rails -T
# rails task 一覧

docker-compose exec web ./bin/rails action_text:install
# action text のインストール

docker-compose exec web ./bin/rails db:migrate
# データ反映 db schema確認

docker-compose exec web bundle install
docker-compose down
docker-compose up
# >> spring,redis,solagraphも一緒に動いた

docker-compose exec web ./bin/rails generate scaffold post title:string

docker-compose exec web ./bin/rails db:migrate

# models -> postsにて
class Post < ApplicationRecord
+  has_rich_text :content
end

# app controllers -> posts_controller.rb
# paramsのコンテント許可を追加
def post_params
  params.require(:post).permit(:title, :content)
  + :content
end

# app views posts _form.html.erb
  コンテントフィールドの追加
+ <div class="field">
  + <%= form.rich_text_area :content %>
+ </div>

# app views posts show.html.erb
  showにコンテント追加
+ <%= @post.content %>

# models -> post.rb
  varidationでtitleの文字数、記入漏れを制限
+ validates :title, length: {maximum: 32}, presence: true

# config -> local.rb(make new file)
  エラーメッセージの日本語化
# config/initializers/locale.rbファイルの内容:
# I18nライブラリに訳文の探索場所を指示する
I18n.load_path += Dir[Rails.root.join('lib', 'locale', '*.{rb,yml}')]
# アプリケーションでの利用を許可するロケールをホワイトリスト化する
I18n.available_locales = [:en, :pt]
# ロケールを:en以外に変更する
I18n.default_locale = :pt

# config -> locales -> ja.ymlを作成
# 下記を追加。yml形式なのでインデント注意
# Interpolationでmodelの数値を引用しています。

ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            base:
              content_attachment_byte_size_is_too_big: 添付ファイルは%{max_content_attachment_mega_byte_size}MB以下にしてください。[%{bytes}/%{max_bytes}]
            title:
              blank: 空欄での投稿はできません。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length}文字を超えています。[%{length}/%{max_content_length}]
              attachments_count_is_too_big: の添付画像の数が%{max_content_attachments_count}個を超えています。
    attributes:
      post:
        title: タイトル
        content: コンテンツ


```
