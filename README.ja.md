# 授業で使える Ansible ロールのラボ：VM でロール、ハンドラー、Vault を学ぶ

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml)

インフラのコード化（Infrastructure as Code）を教えるための、すぐに使える **Ansible ロールのラボ**です。学校や大学が学習者一人ひとりに専用のマシンとブラウザ上のターミナルを提供する [MitsuruTech Learning Platform](https://mitsurutech.co.uk/ja/learning) のラボで、[lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics) の次のステップにあたります。学習者は **ansible-core** が入った Ubuntu の **VM** と小さなプロジェクトを受け取り、**ロール**をゼロから作ります。変数、Jinja2 テンプレート、タグ付きのタスク、一度だけ実行される**ハンドラー**、そして **Ansible Vault** に保管するトークンです。対象はマシン自身なので、ほかに用意するものはなく、学習者のパソコンには何もインストールされません。

## 学習者がすること

ラボは `~/roles-lab` にプロジェクトを、ホームフォルダーに `README.md` を残します。README は 7 つの手順を示しますが、答えは一つも書いていません。

```
ansible.cfg          inventory.ini とロールのフォルダーを指す
inventory.ini        2 つのホスト web1 と spare1、どちらもこのマシン
site.yml             web グループへのプレイ 1 つ。ロール appconf を使う
vault-password.txt   このラボの Vault パスワード
roles/               ロールを置く場所
```

1. プレイブックを実行し、ロール `appconf` がまだ存在しないために止まることを確かめます。
2. `ansible-galaxy role init` でロールを作り、`meta/main.yml` を書き込みます。
3. `defaults/main.yml` と `vars/main.yml` に変数を定義します。`site.yml` はすでにプレイで `app_env` と `app_dir` を設定しているので、どちらが優先されるかを自分で突き止めます。
4. `config` または `user` のタグを付けてタスクを書きます。フォルダー、ループと条件を使ったテンプレートからのファイル、機能ごとに 1 つのファイル、トークンのファイル、システムユーザー、そして小さなファイル 2 つです（うち 1 つはユーザーが本当に作られたときだけ書きます）。
5. 3 つのタスクが通知するハンドラーを一度だけ実行させ、ロールの途中でハンドラーを実行させます。
6. `app_api_token` を `ansible-vault` で作ったファイルに入れます。パスワードファイルで開け、ほかの場所には平文で書きません。
7. 2 回実行し（2 回目は何も変更してはいけません）、そのあと `--check --diff`、`--tags`、`--limit` を試します。

## 採点のしかた

このラボには**チェック**があります。提出された VM を、教員が読まなくても採点するスクリプトです。保存されたマシンのコピーを新しく起動して実行します。まずロールを読みます。標準のフォルダー、書き込まれた `meta/main.yml`、プレイブックではなくロールの中にある 6 つ以上のタスク、ハンドラー、ループ、タグ、そして `for` ループと `if` を使ったテンプレートです。`shell`、`command`、`raw`、`script`（と、Ansible のモジュールを避けるいくつかの方法）、そしてプログラムを実行する `pipe` の lookup は受け付けません。ラボのパスワードで Vault ファイルを開き、プロジェクト内に平文で書かれたトークンがあれば不合格にします。

そのあと、**誰も見たことのない値で学習者自身の `site.yml` を 7 回実行します**。きれいなマシンでの実行、変更するものがない状態での再実行、変更を表示するだけで何も書かない dry run、別の値での実行、タグごとの実行、プレイが使わないホストへの `--limit` です。本物の出力に含まれるハンドラーの実行回数を数えるので、3 回実行されるハンドラーは不合格になり、できあがったファイルも読みます。表示される最初の行が、何が違うかを伝えます。

しないこと：本物のサービスの再起動はしません。チェックでは証明できないからです。また、自分の VM で root 権限を持つ学習者が Ansible そのものを改ざんすることは、防げません。これは授業のためのチェックであり、証明ではありません。

## ファイルの中身

`ansible-roles.lab.yaml` は、プラットフォームがそのまま実行する **VM ラボ定義**の全体です。

| キー | 内容 |
| --- | --- |
| `cpus`, `memory`, `disk` | 学習者ごとの VM のサイズ：CPU 1、メモリ 1024 MB、ディスク 10 GB |
| `open` | ブラウザーで開くポート：なし。ラボはターミナルです |
| `setup` | 各学習者の VM が初めて起動したときに root で一度だけ実行されるスクリプト：Ubuntu のアーカイブから `ansible-core` をインストールし、プロジェクト、プレイブック、偽の Vault パスワード、README を書きます |
| `check` | 採点スクリプト。保存されたディスクのコピーを新しく起動して root で実行します。終了ステータスが結果で、`0` が合格です |

プロジェクトの中の Vault パスワードは**テスト用の固定値**で、秘密ではないとファイル自身に書いてあります。学習者が Vault に入れるトークンもデモ用のデータで、本物の秘密ではありません。

## プラットフォームで使う

**クラスの課題にするには：**

1. 担当するモジュールでタスクを追加し、**Ubuntu VM ラボ**のカードを選びます。
2. Ansible ロールのサンプルを押すか、`ansible-roles.lab.yaml` の内容を貼り付けます。サンプルを使うとテキストが置き換わり、「元に戻す」で自分のものに戻せます。
3. 公開して、まず自分用のコピーで試します。学習者それぞれに専用の VM が用意され、セッションをまたいで保たれます。
4. ほかのタスクと同じように割り当てます。提出されたら、クラス一覧の学習者の行でチェックを実行します。保存されたマシンのコピーが起動し（元には保存されません）、合格したかどうかと出力内容が表示されます。

プラットフォームの Basics レッスンの 1 つは、このラボで取り組めます。マニュアルの[このラボのページ](https://mitsurutech.co.uk/ja/help/lab-ansible-roles)、[Ubuntu VM ラボのガイド](https://mitsurutech.co.uk/ja/help/labs)、[サンプルラボのページ](https://mitsurutech.co.uk/ja/help/sample-labs)に続きがあります。

学校のアカウントがまだありませんか。[最初の 1 年は無料](https://mitsurutech.co.uk/ja/pricing)（学習者 50 人まで）で、[登録](https://mitsurutech.co.uk/ja/signup)にはメールアドレスだけが必要です。

## ファイルを確認する

プッシュのたびに `.github/workflows/check.yml` が、ファイルが上記のキーを持つ有効な YAML であること、`setup` と `check` がそれぞれプラットフォームの 16 KB の上限に収まっていることを確認します。VM は起動しません。それはプラットフォーム自身のマシンで行います。

## ラボでできないこと

VM が届くのは Ubuntu のパッケージアーカイブ、PyPI、npm、GitHub だけで、それ以外には届きません。このラボは Ubuntu のアーカイブだけを使うため、すべての作業は Ansible 自身のモジュールで、ファイル、フォルダー、システムユーザーに対して行います。再起動するサービスはありません。ハンドラーは、チェックに見えるもの、つまり実行された回数と書き込むファイルで示します。対象は VM 自身で、ローカル接続で操作します。

## このラボについて

[Mitsuru Technologies](https://mitsurutech.co.uk/ja) が [MitsuruTech Learning Platform](https://mitsurutech.co.uk/ja/learning) のために作りました。Python、Java、Go、Node、SQL、Linux をブラウザーで学べる、学校・大学向けのプラットフォームで、コンテナラボ、VM ラボ、GitHub から配備する Web アプリ、自動採点のクイズがあります。アカウントなしでコードを試せる[プレイグラウンド](https://mitsurutech.co.uk/ja/playground)もあります。

同じ提供元のほかのラボ：[lab-netlab](https://github.com/mitsurutech/lab-netlab)、[lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop)、[lab-dvwa](https://github.com/mitsurutech/lab-dvwa)、[lab-webgoat](https://github.com/mitsurutech/lab-webgoat)、[lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin)、[lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea)、[lab-sample](https://github.com/mitsurutech/lab-sample)、[lab-sdn-lab](https://github.com/mitsurutech/lab-sdn-lab)、[lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab)、[lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab)、[lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab)、[lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab)、[lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab)、[lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics)。

## ライセンス

この作品は [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.ja) でライセンスされています。Mitsuru Technologies Ltd のクレジットを表示し、商用には使わず、そこから作ったものを同じライセンスで共有する限り、学習と教育のために共有・改変できます。全文は `LICENSE` ファイルにあります。

[Ansible](https://www.ansible.com)（`ansible-core`、GPL-3.0）は VM の起動時に Ubuntu のアーカイブからインストールされます。ここでは何も配布していません。
