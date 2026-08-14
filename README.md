# JudgymPocket

演技動画を見ながら減点ボタンを押すと、押した時刻の 0.5 秒前を切り出して画面下に並べ、Eスコアを自動計算する体操採点アプリ。採点が終わったら、減点スタンプを焼き込んだ動画として書き出せる。

ビルド不要。`index.html` を開くだけで動く単一ファイル構成。

## 使い方

1. 上部から演技動画を選ぶ（スマホのカメラロールから選択可）
2. 再生しながら `-0.3` `-0.5` `-0.1` `-1.0` を押す
3. 押した 0.5 秒前のフレームが下部に並ぶ。サムネイルをタップするとその位置に戻る。右上の × で取り消し
4. Eスコア（10 − 減点合計）は右下にリアルタイム表示
5. 「保存」で結果を確認。そこから「スタンプ入り動画を書き出す」で合成動画を生成し、共有シートから写真アプリに保存できる

右上のメニューから、切り出し位置（既定 0.5 秒前）、スタンプの表示時間、Dスコアの控えを変更できる。

キーボード操作（PC）: `Space` 再生/一時停止、`1`〜`4` 各減点。

## ホーム画面に追加

HTTPS 上で開き、iPhone は共有 → 「ホーム画面に追加」、Android は「アプリをインストール」。
アイコンとアプリ名 **JudgymPocket** が登録され、Service Worker により 2 回目以降はオフラインでも起動する（動画の書き出し処理もすべて端末内で完結し、サーバーには何も送信しない）。

## GitHub Pages で公開する

```bash
git init
git add .
git commit -m "JudgymPocket"
git branch -M main
git remote add origin https://github.com/<ユーザー名>/judgympocket.git
git push -u origin main
```

リポジトリの Settings → Pages → Source を `Deploy from a branch`、Branch を `main` / `(root)` に設定。数分後に `https://<ユーザー名>.github.io/judgympocket/` で公開される。

サブディレクトリ配信を前提に、パスはすべて相対パス（`./`）で書いてある。ルート直下に置く場合もそのままで動く。

## ファイル構成

```
.
├── index.html              アプリ本体（HTML/CSS/JS すべて内包）
├── manifest.webmanifest    アプリ名・アイコン・表示モード
├── sw.js                   オフライン用 Service Worker
├── .nojekyll               GitHub Pages の Jekyll 処理を無効化
└── icons/
    ├── icon-192.png / icon-512.png
    ├── icon-maskable-192.png / icon-maskable-512.png   Android の丸型・角丸切り抜き用
    ├── apple-touch-icon.png                            iOS ホーム画面用（180px）
    ├── favicon.ico / favicon-32.png
    └── icon.svg                                        元データ
```

## 更新するとき

`index.html` を変更したら、`sw.js` 冒頭の `CACHE` の版番号（`judgympocket-v1` → `v2`）を上げる。上げないと古いキャッシュが表示され続ける。

## 動画書き出しについての注意

書き出しは Canvas + MediaRecorder による等倍のリアルタイム合成。45 秒の動画なら書き出しにも 45 秒かかり、再エンコードのぶん多少画質が落ちる。出力形式は端末依存で、MP4 が使えない環境では WebM になる。

無劣化かつ短時間で処理したい場合は、iOS ネイティブの `AVMutableVideoComposition` に置き換えるのが本筋。減点データは `{ value, at, shot }` の配列で保持しているので、そのまま移植できる。
