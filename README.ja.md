# git-stray

[English](./README.md) · **日本語** · [Français](./README.fr.md)

このディスクにしか存在しない作業を見つける。

```
$ git stray ~

git-stray  scanned 165 repositories

AT RISK  ~/courses/swift/Egg
    no remote configured — 1 commit exists only on this disk (last 558d ago)
    7 uncommitted files — untouched for 558d
AT RISK  ~/hackathon/team-20-app
    12 unpushed commits — main(12) (last 336d ago)
STALE    ~/work/old-service
    stash 1922d old — stash@{0}: WIP on feature/settings

2 at risk, 1 stale.
```

## 何をするか

リポジトリの状態を見るツールは「何が変わったか」を答える。
ディスクが飛んだときに本当に必要になる問い ——「**ここにしか無いものは何か**」——
に答えるものは無い。

push していないブランチ、remote を設定し忘れたリポジトリ、5年前の stash。
どれも壊れてはいないので、何も警告してこない。ローカルでは全て正常なので
`git status` は黙っている。単に、コピーが1つしか存在しないだけ。

git-stray は指定パス配下の全リポジトリを走査して、それだけを報告する。

## やらないこと

**プロジェクトが古いことは報告しない。**

放置は問題ではない。8ヶ月触っていないのはたいてい予定の問題であって、
ミスではない。静かなプロジェクトを並べ立ててくる道具は、いずれ消される。

そのため未コミット変更と stash は `--days`（既定14日）を超えたものだけを報告し、
活動量が理由で何かが報告されることは無い。未 push コミットと remote 無しの
リポジトリは常に報告する。こちらは本当にコピーが1つだから。

## インストール

```sh
curl -o /usr/local/bin/git-stray \
  https://raw.githubusercontent.com/urabexon/git-stray/main/git-stray
chmod +x /usr/local/bin/git-stray
```

`PATH` 上の `git-*` は git のサブコマンドになるので、`git stray` で呼べる。
必要なのは bash 3.2 以降（macOS 標準の bash で可）と git。

## 使い方

```sh
git stray                  # カレントディレクトリ配下
git stray ~                # ホーム全体
git stray --days 30 ~      # 1ヶ月より古いものだけ
git stray --days 0 ~       # 新しいものも全部
git stray --fetch ~        # remote ref を更新してから比較（ネットワーク・遅い）
git stray --hidden ~       # 隠しディレクトリ配下も含める
git stray --depth 6 ~      # 探索の深さ（既定 4）
```

終了コードは `0` = コピーが1つのものは無い、`1` = ある。

```sh
git stray ~ || echo "閉じる前に何か push しろ"
```

## 報告する内容

| 重大度 | 内容 |
|----------|-------|
| AT RISK | どのリモート ref からも到達できないローカルブランチのコミット |
| AT RISK | 同上、detached HEAD のコミット |
| AT RISK | remote が1つも無いリポジトリ（全コミットがコピー1つ） |
| STALE | 追跡ファイルの未コミット変更で `--days` より古いもの |
| STALE | `--days` より古い stash |

隠しディレクトリ（`~/.nvm` など）配下は既定で除外する。他人の clone であって
自分の作業ではないため。

日数はコミット日・stash 日・ファイルの mtime から求める。削除やリネームのように
mtime を読むファイルが無い場合は最終コミット日で近似し、`~244d` のように印を付ける。
**黙って落とすことはしない。**

## 精度について

比較対象はディスク上の remote-tracking ref なので、高速かつオフラインで動くが、
最後に `fetch` した時点の情報でしかない。厳密にやるなら `--fetch` を使う
（リポジトリごとに1往復する）。

「未 push」は**すべてのリモート ref に対して**計算する。upstream 未設定のブランチや、
別名で push されたブランチも、正しく push 済みとして扱われる。

## 関連

- [git-brink](https://github.com/urabexon/git-brink) — 兄弟ツール。
  brink は**出てはいけないものが出る**のを止め、stray は**出るべきものが出ていない**のを見つける
- [gita](https://github.com/nosarthur/gita) / [mr](https://myrepos.branchable.com/) —
  複数リポジトリの一括操作。全部を見せるのが目的で、危ないものだけを抜くこちらとは役割が違う
- [gitleaks](https://github.com/gitleaks/gitleaks) — コミット済みの内容から秘密情報を探す。別問題

## ライセンス

[MIT](./LICENSE)
