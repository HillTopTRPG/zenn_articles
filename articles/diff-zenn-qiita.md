---
title: "Zenn vs Qiita 書きやすさ比較"
emoji: "⚖️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Zenn", "Qiita"]
published: true
---

# Zenn vs Qiita 書きやすさ比較

とりあえず両者ともに初投稿してみたはいいものの  
今後も投稿していくとしたら、どっちにしようかな？という話

# 比較してみた

触ってみて気づいたポイントを雑ですがまとめました。  
優勢の判断は各サービスの特徴の違いを踏まえた個人的な主観によるもので、  
絶対的な評価ではないことをご留意ください。

| # | 比較項目           |   Zenn   | Qiita | 優勢 |
|:--|:---------------|:--------:|:--------:|:--:|
| 1 | 投稿形式           | 記事, 本 ※1 | 記事 | \> |
| 2 | フォーマット         | Markdown | Markdown | ≒  |
| 3 | CLI & GitHub連携 |    良     |    良     | ≒  |
| 4 | 限定公開           |   ない   |    ある    | \< |
| 5 | 記事URL          |  任意  | UUIDのみ | \> |

※1 スクラップも選べるが、CLIから投稿できないため除外とした

## 1. 投稿形式

Zennは本という形式でも投稿できるらしいです。  
値段をつけることで収益化も可能とのこと。

https://zenn.dev/zenn/books/how-to-create-book

複数の記事を章立てでまとめておけるらしいです。  
長い取り組みの忘備録を段階ごとに記録するのにも使えるかもしれませんが、  
普通に記事間のリンクで連続性を表現する方が良いかもしれないです。

使い所は考える必要がありますが、選択肢があるのはありがたいです。

## 2. フォーマット

どちらも Markdown記法で記述します。  
差異があるかは未確認ですが、少なくとも執筆上困ることはなさそう。

https://zenn.dev/zenn/articles/markdown-guide

https://qiita.com/Qiita/items/c686397e4a0f4f11683d

## 3. CLI & GitHub連携

両者とも、指定したリモートブランチへのコミットによる記事の公開・更新が可能でした。  

https://zenn.dev/zenn/articles/connect-to-github

https://qiita.com/Qiita/items/32c79014509987541130

QiitaはGitHub Actionsによる実装（ゆえにアクセストークンが必要）
GitHub Actionsゆえに、進行状況やエラーなどはGitHub上で確認できます。  

Zennは逆にどういう実装になっているんでしょうね。  
アクセストークンが必要ない分、管理する人は楽かもしれません。  
進行状況やエラーはダッシュボードから確認できるようです。  

個人的にはどちらも良い体験だったなと思っています。

## 4. 限定公開

Zennの記事のステータスは、`下書き` → `公開` となっており、  
Qiitaでは `限定公開` or `全体に公開` という感じでした。  

Zennの `下書き` は筆者しか閲覧できませんでした。  
Qiitaの `限定公開` はURLさえ知っていれば誰でも見ることができました。  

限定記事はクローラー対策されていました。

``` html:限定記事に含まれていたメタタグ
<meta content="noindex, nofollow" name="robots">
```

比較表の上では限定公開ができるということで Qiita を優勢にしましたが、  
人の好みによるかなという部分だと思います。

## 5. 記事URL

記事のURLがどうなるかが両者で違いました。  

検証した記事（初投稿）は下記の通りです。

https://zenn.dev/hilltop/articles/started-zenn

https://qiita.com/HillTopTRPG/items/3fbf896a7cf8d0ee6231

記事を一意に表す部分が、Zennでは `started-zenn` なのに対し、  
Qiitaでは `3fbf896a7cf8d0ee6231` なのがお分かりいただけると思います。  

この違いを説明するため、まずはCLIで執筆した生のファイルをご覧いただきたいと思います。  
※ 項目の意味を示すためのコメントを付与しています

```yml:started-zenn.md
---
title: "Zenn、はじめました"
emoji: "🐣" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Zenn"]
published: true # 公開設定（falseにすると下書き）
---

記事のコンテンツ
```

```yml:started-qiita.md
---
title: Qiita、はじめました
tags:
  - Qiita
private: false # true: 限定共有記事 / false: 公開記事
updated_at: '2024-08-29T11:23:46+09:00' # 記事を投稿した際に自動的に記事の更新日時に変わります
id: 3fbf896a7cf8d0ee6231 # 記事を投稿した際に自動的に記事のUUIDに変わります
organization_url_name: null # 関連付けるOrganizationのURL名
slide: false # true: スライドモードON / false: スライドモードOFF
ignorePublish: false # true: `publish`コマンドにおいて無視されます（Qiitaに投稿されません） / false: `publish`コマンドで処理されます（Qiitaに投稿されます）
---

記事のコンテンツ
```

どちらのファイルも `started-[サービス名].md` というファイル名です。  
Zennはファイル名がそのままURLの一部となります。  
Qiitaでは `id` の項目があり、それがURLの一部となります。

個人的には、Zennの仕様の方が好きです。

# まとめ

現状の判断だと、**Zennがやや優勢**という感じです。

まだ１記事ずつしか作成・公開していない状態での比較ですので、  
今後の利用で比較・検証すべきポイントが増えるかもしれません。  
