# データサイエンス J26

Jupyter Book 2 / MySTで作成したデータサイエンスの授業教材。授業ガイダンスはトップページ [index.md](index.md)、第1回は [lesson-01.md](lesson-01.md)、第2回は [lesson-02.md](lesson-02.md) にある。第1回の演習ノートブックとPyMCによる分類の発展ノートは、それぞれ独立した子ページとして提供する。

## ローカルで読む

```bash
uv sync
uv run jupyter book start
```

HTMLを出力する場合は `uv run jupyter book build --html --strict --execute` を使う。

表示された `http://localhost:3000` をブラウザーで開く。

## 演習ノートブックの再実行

```bash
uv run jupyter execute exercises.ipynb --inplace
```

図中の日本語フォントには `matplotlib-fontja` を使用する。

## GitHub Pages

`main` へのpushで `.github/workflows/deploy.yml` がHTMLをビルドしてGitHub Pagesへデプロイする。リポジトリ設定の **Settings → Pages → Build and deployment → Source** を **GitHub Actions** にする。
