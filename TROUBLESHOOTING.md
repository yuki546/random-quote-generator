# トラブルシューティング

## GitHub Actions デプロイエラー: `npm ci` の失敗

### 症状

GitHub Actions で以下のようなエラーが発生する：

```
npm error `npm ci` can only install packages when your package.json and package-lock.json or npm-shrinkwrap.json are in sync.
npm error Missing: @emnapi/core@1.9.2 from lock file
npm error Missing: @emnapi/runtime@1.9.2 from lock file
```

### 原因

package-lock.json が破損または不完全な状態になっている。特定のパッケージ（例: `@emnapi/core`, `@emnapi/runtime`）への参照は存在するが、それらのパッケージ自体の定義（`node_modules/@emnapi/core` エントリなど）が欠落している。

この問題は以下の場合に発生しやすい：
- npm のバージョンが異なる環境間でインストールを繰り返した
- 依存関係の更新中に中断された
- git マージ時に package-lock.json が不完全な状態になった

### 解決方法

#### 手順 1: package-lock.json を完全に再生成

```bash
# node_modules と package-lock.json を削除
rm -rf node_modules package-lock.json

# クリーンインストール
npm install
```

#### 手順 2: npm ci でテスト

```bash
# GitHub Actions と同じコマンドでテスト
npm ci
```

エラーが出なければ成功です。

#### 手順 3: ビルドのテスト

```bash
npm run build
```

ビルドも成功することを確認します。

#### 手順 4: 修正を Git にコミット

```bash
git add package-lock.json
git commit -m "package-lock.json を再生成してデプロイエラーを修正"
git push
```

### 予防策

1. **npm のバージョンを統一する**
   - プロジェクトで使用する Node.js と npm のバージョンを `.nvmrc` や `package.json` の `engines` フィールドで指定する
   - GitHub Actions の Node.js バージョンとローカルのバージョンを一致させる

2. **package-lock.json を常にコミットする**
   - `.gitignore` に `package-lock.json` を含めない
   - 依存関係を更新したら必ず package-lock.json もコミットする

3. **定期的に npm ci でテストする**
   - デプロイ前にローカルで `npm ci` を実行して問題がないか確認する
   - CI/CD と同じコマンドを使うことで、デプロイ前に問題を発見できる

4. **依存関係の更新は慎重に**
   - `npm install` で個別のパッケージを追加・更新した後は、必ず `npm ci` でテストする
   - 更新後は package-lock.json の変更を確認して、意図しない変更がないかチェックする

### 関連情報

- [npm ci のドキュメント](https://docs.npmjs.com/cli/v10/commands/npm-ci)
- GitHub Actions で使用している Node.js バージョン: [.github/workflows/deploy-gh-pages.yml](.github/workflows/deploy-gh-pages.yml#L36) を参照

### 発生履歴

- 2026-04-16: @emnapi/core, @emnapi/runtime の定義欠落による npm ci エラー
  - 解決方法: package-lock.json の完全再生成
