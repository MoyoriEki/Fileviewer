# Fileviewer (Dropbox Filer)

Dropbox 上の画像・ファイルを閲覧する Web ビュアー（`moyorieki.github.io` 配下・同一オリジンで [プロットボード](https://github.com/MoyoriEki/PlotBuilder) と連携）。単一ファイル `index.html`。

## 変更履歴

- **Phase 5（.clip サムネイル + サムネキャッシュ改善）**: CLIP STUDIO PAINT の `.clip` をグリッドでサムネイル表示する。Dropbox の `get_thumbnail_v2` は `.clip` 非対応なので、ファイル内部から自前で取り出す。`.clip` は `CSFCHUNK` コンテナで、`8バイトのチャンク名 + ビッグエンディアン u64 の長さ + データ` が並ぶ構造。その中の `CHNKSQLi` が素の SQLite データベースで、`CanvasPreview` テーブルにキャンバス全体の PNG が入っている。SQLite は依存を増やさないよう最小限の読み取り専用パーサを内蔵（b-tree 走査 + オーバーフローページ連結）。WebAssembly 版 SQLite を使わないのでオフライン動作と単一ファイル構成を維持できる。ダウンロードは `/files/download` の Range リクエストで、ヘッダのオフセットヒント → 末尾 2MB スキャン → チャンク走査 → 広域スキャンの順に安く探す。取り出した PNG は長辺 512px の JPEG に縮小してから保存する。非公開フォーマットなので各段階を検証し、解釈できないファイルは静かにアイコン表示へフォールバックする。
  あわせてサムネキャッシュを改善。キャッシュキーの世代識別子を `content_hash || rev || server_modified` にして、一覧が `content_hash` を返さない場合でも保存のたびに更新されるようにした。新設の `thumbMeta` ストアがパスごとの最新キーを持ち、世代が変わったときは旧サムネを即座に表示してから裏で差し替える（stale-while-revalidate）。差し替え成功時に旧キーを削除するので、頻繁に上書き保存される `.clip` でも IndexedDB が際限なく太らない。IndexedDB は v3。古いタブが残っていてアップグレードがブロックされた場合も `init()` を止めずキャッシュ無効で起動する。

- **Phase 4.5（プロットボード連携・D&D 画像添付）**: サムネイルのラッパー（`.grid-item`）ドラッグ時、ファイルなら `text/plain` に Dropbox 原本パスの JSON（`{v:1, source:'fileviewer', dropboxPath, name}`）を載せて送出。別窓のプロットボードがこのパスを受け取り、自力でダウンロード→縮小→添付する（File 実体はブラウザ制約で別窓へ渡せないため文字列で受け渡す）。サムネ `<img>` は `draggable="false"`（既定の画像ドラッグが自前ペイロードに混ざるのを防ぐ）。`effectAllowed` は内部移動の `move` と外部窓への `copy` を両立させる `copyMove`。内部の移動・並べ替え D&D 挙動は従来どおり（`dragSrcIdx` で処理し `text/plain` は読まない）。PC の 2 窓運用向け。
