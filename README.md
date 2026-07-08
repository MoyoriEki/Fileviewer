# Fileviewer (Dropbox Filer)

Dropbox 上の画像・ファイルを閲覧する Web ビュアー（`moyorieki.github.io` 配下・同一オリジンで [プロットボード](https://github.com/MoyoriEki/PlotBuilder) と連携）。単一ファイル `index.html`。

## 変更履歴

- **Phase 4.5（プロットボード連携・D&D 画像添付）**: サムネイルのラッパー（`.grid-item`）ドラッグ時、ファイルなら `text/plain` に Dropbox 原本パスの JSON（`{v:1, source:'fileviewer', dropboxPath, name}`）を載せて送出。別窓のプロットボードがこのパスを受け取り、自力でダウンロード→縮小→添付する（File 実体はブラウザ制約で別窓へ渡せないため文字列で受け渡す）。サムネ `<img>` は `draggable="false"`（既定の画像ドラッグが自前ペイロードに混ざるのを防ぐ）。`effectAllowed` は内部移動の `move` と外部窓への `copy` を両立させる `copyMove`。内部の移動・並べ替え D&D 挙動は従来どおり（`dragSrcIdx` で処理し `text/plain` は読まない）。PC の 2 窓運用向け。
