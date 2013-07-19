---
layout: post
title: "Octopressとgithub pagesをつかってブログをつくる"
date: 2013-04-14 13:06
comments: false
categories: [Github]
sharing: true
footer: true
keywords: [Github]
---
最近よく`username.github.com/blog/2013/04/13/title`といったuriのブログを見かけるようになった。  
`username.github.com`がかっこいいから作りたくなったので作ってみた。  

## このブログは何？
下の方に書いてる`Powered by Octopress`のとおり[Octopress](http://octopress.org/)で作っています。 
公式サイトに書いてる通りにやったらできました！  

## 必要なもの
 * 興味！
 * ruby
 * githubアカウント

## やったこと

### ブログ用リポジトリ作成
これはgithubの機能。  
この辺 [https://help.github.com/categories/20/articles](https://help.github.com/categories/20/articles)をみると詳しく書いてた。  

1. [https://github.com/new](https://github.com/new) から`username.github.com`という名前のリポジトリを作る。
2. index.htmlをpushする。

```sh
# このブログの場合
touch index.html
git init
git add index.html
git commit -m "first commit"
git remote add origin https://github.com/sohda/sohda.github.com.git
git push -u origin master
```

ここまでやったらコーヒータイム。10分くらい休憩。  
休憩したら`username.github.com`にアクセスしてみる。  
とindex.htmlに書いた内容が表示されているはず！  
何も書いてなければ真っ白な画面。  
404がでたらもう少し休憩。  
githubの機能を使うので反映されるまで10分くらいかかるそうです。  

### Octopress インストール
```
cd $HOME
git clone git://github.com/imathis/octopress.git octopress
cd octopress
bundle install
rake install
rake setup_github_pages
rake generate
rake deploy
git add .
git commit -m 'your message'
git push origin source
```
ここまでやったら、作ったリポジトリをみにいくとsourceブランチにソースコードがpushされています。

### 投稿してみる
さっそく投稿。の前にすこしだけ設定。
```
$ vi _config.yml
$ cat _config.yml
# 中略 #
url: http://sohda.github.io
title: saber_chica Alog
subtitle: "saber_chica's blog"
author: Tomoko Sohda
simple_search: http://google.com/search
description:
# 中略 #
permalink: /tomoko/:year/:month/:day/:title/
# 中略 #
```

設定が終わったら
```
$ rake gen_deploy
```

記事作成！
```
$ rake new_post["test post"]
mkdir -p source/_posts
Creating new post: source/_posts/2013-04-13-test-post.markdown
vi source/_posts/2013-04-13-test-post.markdown
```

ブログの内容はここに書く
```
---
layout: post
title: "test post"
date: 2013-04-13 16:40
comments: true
categories: [Github]
sharing: true
footer: true
keywords: [Github]
---
# ココからブログの本文
```

いよいよ投稿！の前に確認も！
```
$ rake preview
$ rake gen_deploy
```

ここまでできたらもう1回`username.github.com`にアクセス!  
今の投稿が表示されてる！  
  
ブログが文字化けしてる場合はファイルのエンコードがutf-8になってないかも

```
vi $HOME/.vimrc
# いかの設定にutf-8を指定しておく
set enc=utf-8
set fenc=utf-8
set fencs=utf-8,....
```

----

#### 参考URL
* [http://octopress.org/docs/setup/](http://octopress.org/docs/setup/)
  * ruby2.0-p0で動いてくれました！
