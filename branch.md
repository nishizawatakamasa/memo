# GitHubでの作業マニュアル（参考程度に）

## コミットメッセージの書き方の例
* docs: ドキュメントのみの変更
* refactor: リファクタリング
* feat: 機能の追加や変更
* fix: バグの修正
* test: テストコードの追加や修正
* chore: プロダクションに影響のない修正

## ブランチ名
* 永続ブランチ → main（メイン）
* 一時ブランチ → feature（フィーチャー）
* 編集者名付き一時ブランチ → name-feature（編集者名-フィーチャー）

## 操作コマンドでブランチを切ってマージするまでの流れ
1. `git branch feature` ローカルにfeatureブランチを作成

1. `git checkout feature` featureブランチへ移動

1. `git push -u origin feature` ローカルにあるfeatureブランチをリモートにも上げる

1. `git add .` 変更点をステージ

1. `git commit -m "コミットメッセージをここに書く"` ステージされている変更をローカルリポジトリに登録

1. `git push` ローカルリポジトリに登録されている変更をリモートリポジトリに反映

1. `git checkout main` mainブランチへ移動

1. `git fetch --all` 念のため最新のリモートコミットを取得

1. `git merge feature` mainブランチへfeatureブランチをマージする

1. `git push` マージした結果をリモートリポジトリに反映

1. `git branch -d feature` ローカルにあるfeatureブランチを削除する ※現在の作業ブランチを削除することはできない

1. `git push origin --delete feature` リモートにあるfeatureブランチを削除する

## おまけ：便利コマンド
* `git checkout -b feature` featureブランチを作成して、featureブランチへ移動

* `git branch` ローカルブランチ一覧確認（*印が現在いるブランチを示す）

* `git branch -a` リモートブランチ一覧確認（*印が現在いるブランチを示す）

## リモートURL関連
* `git remote -v` 既存のリモートURLの設定を確認する

* `git remote set-url origin 変更したいURL` 新しいリモートURLに変更する