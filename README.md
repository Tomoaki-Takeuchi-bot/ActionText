# Rails6 Action Text
### このアプリはRails6の新機能であるActiontextの動作検証を行った簡易アプリです。

![ActionText](https://drive.google.com/file/d/1lgELTCoXyOWkiQZwpetknE-BQuhrrEFP/view?usp=sharing.ActionText.png)

- 作成目的
  - Docke,Docker-composeでの安定的な環境構築（Dockerfile,docker-compose.yml,qs )
- アプリ追加仕様
  - ActionText
  - validation (title文字数、content文字数、添付ファイルサイズ、添付ファイル数)
  - validation error(日本語化)

### 仕様
- Rails: 6.0.3.2
- Ruby: 2.6.6
- Docker/Docker-compose
- ActionText


<インストール手順>
---
### action text のインストール & migrate

```
docker-compose exec web ./bin/rails action_text:install
docker-compose exec web ./bin/rails db:migrate
# データ反映はdb schemaを確認してください。
```

#### Gemfile修正, bundle, 再起動

```
gem 'image_processing', '~> 1.2'
# 上記がコメントアウトされているので有効に変えてください。
docker-compose exec web bundle install
docker-compose down
docker-compose up -b web db
# docker-compose は -b web db で起動してください。
# 入れない場合他のコンテナも起動してしまいます。
```

### scaffold post & has_rich_textをモデルに追加

```
docker-compose exec web ./bin/rails generate scaffold post title:string
docker-compose exec web ./bin/rails db:migrate

# models -> postsにて + 行を追加してください。（+に関する説明は以下同様とします。）
class Post < ApplicationRecord
+  has_rich_text :content
end
```

### paramsのコンテント許可を追加

```
# app controllers -> posts_controller.rb
def post_params
  params.require(:post).permit(:title, :content)
  + :content
end
```

### コンテントフィールドの追加

```
# app views posts _form.html.erb
+ <div class="field">
  + <%= form.rich_text_area :content %>
+ </div>
```

### showにコンテント追加

```
app views posts show.html.erb
+ <%= @post.content %>
```

### varidationでtitleの文字数、記入漏れを制限
#### *詳細コードはmodel -> post.rbを参照願います。
```
#  必要に応じて下記のコードを変更しtitleの長さや、添付ファイルの制限を変更してください。
validates :title, length: { maximum: 40 }, presence: true 
  MAX_CONTENT_LENGTH = 100
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 4
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE
  MAX_CONTENT_ATTACHMENTS_COUNT = 4

```

### エラーメッセージの日本語化
#### *railsガイドからの記述に沿ってymlファイルを追加しています。

```
# config -> local.rb(make new file)
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

