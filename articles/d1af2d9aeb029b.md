---
title: "祇園祭Webサイト2024のサーバ周りについて"
emoji: "🐟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["almalinux", "script", "さくらのvps", "cloudflare"]
publication_name: nitkc_proken
published: true
---

## なにこれ
この記事は [木更津高専 Advent Calender 2024](https://qiita.com/advent-calendar/2024/nit_kisarazu) 参加記事です．（大遅刻です．ごめんなさい．）
前日の記事は [@naotiki](https://qiita.com/naotiki) さんによる [今年もプロ研が学園祭Webサイト制作をした話](https://qiita.com/naotiki/items/89e18320b8ec20dd91cb) でした．
翌日の記事は [@nairoki](https://qiita.com/nairoki) さんによる [Minio×Hono×Reactをやろうとした経験談を書いておこうと思ふ。](https://qiita.com/nairoki/items/54996e6241bbab1c2b84) です．

今年度も，プロ研は祇園祭の Web サイトを制作しました．  
私はサーバ関連を担当したため，その記録を残しておこうと思います．

## 環境について
昨年度 Web サイトを制作・運用した感じではスペック面に問題がなさそうだったため，昨年度と同じ構成で環境を構築しました．

### サーバ周り
サーバは [さくらのVPS 2G](https://vps.sakura.ad.jp/) を使用しました．  
運用中にスペック面の問題が発生したらスケールアップできるように，Nagios を用いて監視環境を構築しました．（後述）  

OS には RHEL 系であることと，私自身の使用経験が長いことから Alma Linux を選択しました．

### ドメイン周り
ドメインは お名前.com を用いて取得し，ネームサーバに Cloudflare を使用しています．  
比較的簡単に SSL 対応ができて便利です．

また，https://gionsai.jp を昨年度のアーカイブ版 (https://2023.gionsai.jp) へリダイレクトする設定を解除し，契約した VPS の IP アドレスへ向くように設定しました．

### ソフトウェア周り
私は以下のソフトウェアを導入・設定しました．  
昨年度は（確か）一人で導入作業を行いましたが，今年度は導入すべきソフトウェアが増えたことやミスが怖かったので [@naotiki](https://qiita.com/naotiki) 氏と一緒に Discord の VC を用いて画面共有をしつつ共同で作業を行いました．

| 名称       | 用途                   |
| ---------- | ---------------------- |
| Firewalld  | ファイアウォール       |
| Nginx      | リバースプロキシ       |
| Certbot    | SSL 証明書の発行       |
| MinIO      | オブジェクトストレージ |
| PostgreSQL | データベース           |
| NCPA       | サーバ監視クライアント |

## 作業ログの管理に悩む
昨年度はサーバ上での作業ログを取っていなかったので，今年度はしっかりログを残そうと考えました．  
当初は Discord 上にログを貼るためのスレッドを立てて都度コピペしていましたが，これが地味に面倒で，コピペ忘れも起こりそうなので，なんとかならないかと悩んでいました．

そこでググって見つけたのが`script`コマンドです．  
これを使えばコマンドの実行履歴・結果をそのままテキストとして書き出すことができます．

ホームディレクトリ直下に作業ログ保管用ディレクトリを用意して，作業内容ごとにファイルを分けて保存することで「ログのコピペ面倒すぎ問題」は一応解決しました．  
ただし，`vi`等のビジュアルエディタを使うと`cat`で内容を出力した時に表示が崩れてしまうため，ファイル編集後に`diff --color`を使って変更箇所を明示する方法をとりました．（`ed`を使うと表示が崩れないという事実が判明したものの，つらい...）

来年度は`script`コマンドの発行やファイル編集時の`diff`出力を自動化できたらいいなーと思っています．

## Nagios を用いたサーバ監視システムの構築
私自身，今年度の夏にとある会社のインターンに参加してサーバ監視について学んだため，早速実践してみました．

Nagios サーバ自体は私の自宅環境を監視するために Oracle Cloud Infrastructure のインスタンス上に構築してあったため，それを使って監視環境を構築しました．  
監視対象には NCPA を導入して，以下の内容について監視しました．

- CPU 使用率
- `/`のディスク使用率
- メモリ使用率
- プロセス総数

また，Discord へ通知を飛ばすシステムは自作するつもりでしたが，余裕がなかったため外部のスクリプトを使用しました．
https://exchange.nagios.org/directory/Plugins/Notifications/Discord-Notifications/details

来年度は更に上位のレイヤ（アプリケーションレベル）の監視をしたいと思います．

## さいごに
今年度も関係者の皆様に助けられ，大きなトラブルなくWebサイトを運営することができました．  
今後ともプロ研をよろしくお願いします．