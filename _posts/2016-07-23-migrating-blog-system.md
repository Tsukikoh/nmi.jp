---
layout: post
title: WordPressからJekyllにブログシステムを移行
---
このサイトですが、WordPressのあまりの重さに耐えかねて、ついにJekyllに移行しました。WordPress自体は大変良いものだという印象を持っているのですが、それと引き換えにシステムが大変重くてブログ記事を書くモチベーションが大きく下がっており、これじゃ本末転倒だということで移行を決断しました。




実際の移行は[グニャラくんの記事](http://blog.wktk.co.jp/ja/entry/2013/04/26/wordpress-to-jekyll)を参考にしました。この記事に書かれていなかったところでは、

- 移行のために `jekyll-import` を使ったところ、コメントの絵文字で文字コードのコンバートに失敗して落ちていたので、importerのソースをいじってコメントをexportしないようにした（そもそもコメント欄は何故か荒れていたのでちょうど良かった）
- importerのソースをいじってパーマリンクを記事ごとに正確に設定し、新しいシステムでもURLの変更が無いようにした

くらいですね。

自分はJekyllについてほとんど調べずに突っ込んでいったので、普通の人が当たり前に知っていることを知らずに幾つか失敗したので、Jekyllの常識をここに書いておきます。

- Jekyll は、基本的に `_posts/` ディレクトリの下にあるファイルを自動的にhtmlに変換してくれる
- `_posts/` 以下のファイルは、 `YEAR-MONTH-DAY-TITLE.ext` というファイル名が必須である。それが設定によって適切なpermalinkに変換される
- 各postに、YAMLによってpostそれぞれの設定を書くことが出来る。たとえば記事ごとの個別のpermalinkはここで設定する（この記事と前の記事でpermalinkが大きく違っているのはこのせい）
- 未来の日付はpublishされない
- 変換先は `_sites/` ディレクトリになる。 `jekyll serve --watch` で自動ビルド＆localhostで確認出来る
- 静的ファイルは適当に配置したら自動的に `_sites/` 以下に展開してくれる

使ってみてわかったのですが、これだけシンプルだと使いやすくて便利ですね。移行にもっと時間がかかるかと思っていたけど、予想よりは随分と早く移行が完了しました。

これで[github](http://github.com/tkihira/nmi.jp)から直接記事を書くことが出来るようになったし、今後記事を書くペースを上げていきたいと思います。

---------------------
追記:
なんちゃってgithub連携の自動更新機能を追加しました。とにかく手軽にやりたかったので、かなり手を抜いています。

サーバ側でこんなスクリプトを書きました。

```sh
#!/bin/sh
fsize=`wc -c _update_check | awk '{print $1}'`
if [ $fsize -gt 1 ]
then
  git pull
  /home/admin/.rvm/gems/ruby-2.2.4/wrappers/jekyll build
  cp /dev/null _update_check
fi
```

要は `_update_check` ファイルのサイズが1より大きければ、pullしてbuildして `_update_check` を空にするスクリプトです。こいつをcronで10秒おきに回しています。

そしてPHPスクリプトで

```php
<?php
shell_exec("echo updated > /path/to/_update_check");
```

というのを書きました。何で書いても良かったんですけど、CGIとかの設定しなくても動くのが楽だったのでPHP選択。そして最後にgithubのWebhooksにこのPHPのURLを設定し、`_update_check`のパーミッションを適切に設定して完了です。

お手軽！
