---
name: add-game
description: hakai-games リポジトリに新しいブラウザゲームを追加する際の作業フロー。ゲーム本体の配置、一覧画面・README の更新、push 前の動作確認をひとまとめにする。
---

# add-game

このリポジトリ（hakai-games）に新しいゲームを追加するときの定型フロー。
**`main` への直 push が deploy** なので、必ずローカルで動作確認してから push する。

## トリガー

- 手動: `/add-game`
- 自動: 「新しいゲームを追加して」「ゲームを実装して」「<ゲーム名> を作って」等

## 必須チェックリスト

新ゲーム追加時は以下をすべて満たすこと。

### 1. ゲーム本体を `docs/` 配下に配置する

- パス: `docs/games/<game-name>/index.html`
- **1ファイル完結**（inline `<style>` / inline `<script>`）。外部 CSS/JS や CDN, npm 依存は禁止。
- 必須要素:
  - `<meta name="viewport" content="width=device-width, initial-scale=1.0" />`
  - モバイル対応（pointer events / `touch-action: none` / 大きめタップ領域）
  - 一覧画面に戻るリンク `<a href="../../index.html">` を必ず置く
  - プレイヤー向け文言は **日本語**

### 2. ゲーム一覧画面を更新する

- 対象: `docs/index.html` の `<table class="games">` 内 `<tbody>`
- 既存末尾の次の番号（ゼロ埋め3桁、例: `005`）で `<tr>` を1行追加
- 列構成:
  - `<td class="num">` … 連番
  - `<td><a href="./games/<game-name>/">タイトル</a></td>`
  - `<td class="desc">` … 短い日本語の説明

### 3. 関連ドキュメントを更新する

- `README.md` の「Play」節に新ゲームの URL 行を追加
  （形式: `- <タイトル>: \`https://hirakumori.github.io/hakai-games/games/<game-name>/\``）
- 仕様書から実装した場合は `design-docs/` 配下のファイルへの参照も「Game Design Docs」節に追加
- `CLAUDE.md` に書かれている既存ゲーム一覧の説明と矛盾が出たら、そちらも更新する

### 4. push 前にローカルで動作確認する

これは必須。「コードを書いた → push」だけで終わらせない。

```sh
python3 -m http.server --directory docs 8000
```

を起動し、以下をブラウザで確認:

- [ ] `http://localhost:8000/` を開き、一覧に新ゲームの行が出ている
- [ ] 行のリンクから新ゲームページに遷移できる
- [ ] ゲームのコア操作（プレイ／勝敗／スコア更新）が一通り動く
- [ ] モバイル幅（DevTools のレスポンシブ表示）でレイアウトが崩れない・タップ操作が効く
- [ ] 「← 戻る」リンクで一覧画面に戻れる
- [ ] DevTools の Console にエラーが出ていない

確認できない不具合がある場合は push しない。確認できなかった項目があれば、ユーザーに「ここは未確認」と明示する。

### 5. コミット & push

- コミットは小さく、メッセージは英語で要点 + （日本語 description でも可）
- `main` に直接 push する（hobby repo なので PR は不要、ユーザーが明示要求した時のみ PR）

## やってはいけないこと

- `docs/` の外にゲームファイルを置く（GitHub Pages の deploy ルートが `/docs` なので公開されない）
- npm / CDN / フレームワーク依存を入れる
- `docs/games/penguin-blocks/index.html` の bundler 出力を手で書き換える（opaque 扱い）
- 公式 Tetris 由来の名称・配色・ロゴ・効果音を使う（README で非関連を明言しているため）
- 動作確認を飛ばして push する
