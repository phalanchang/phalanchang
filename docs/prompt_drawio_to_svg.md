# drawioファイルをSVGに変換するためのプロンプト

## プロンプト

```
以下の手順で、drawioファイルをSVGに変換してください。

### 前提
- 環境: WSL2 (Ubuntu)
- drawioファイルのパス: <drawioファイルの絶対パス>
- SVG出力先のパス: <出力先SVGの絶対パス>

### 手順

1. **draw.io デスクトップ版と xvfb のインストール**
   - draw.io の最新 .deb パッケージを GitHub Releases からダウンロードしてインストール
     - URL例: https://github.com/jgraph/drawio-desktop/releases/latest
     - `sudo dpkg -i <パッケージ>.deb && sudo apt --fix-broken install -y`
   - xvfb をインストール: `sudo apt-get install -y xvfb`
   - draw.io のバイナリは `/opt/drawio/drawio` にインストールされる

2. **drawioファイルのエクスポート前の修正**
   - drawioファイルの `mxGraphModel` 要素にある `shadow="1"` を `shadow="0"` に変更する
   - スタイル属性で `strokeColor:#` や `fillColor:#` のようにコロンが使われている箇所を `strokeColor=#`, `fillColor=#` のように `=` に修正する

3. **SVGへのエクスポート**
   - 以下のコマンドでSVGに変換する:
     ```bash
     xvfb-run -a /opt/drawio/drawio --export --format svg \
       --output <出力先SVGパス> \
       <入力drawioファイルパス> \
       --no-sandbox
     ```
```

---

## 注意事項・トラブルシューティング

### 1. npm の `drawio` パッケージと draw.io デスクトップは別物
- `npm install -g drawio` でインストールされる `drawio`（v1.0.7）はCSV変換ツールであり、図のエクスポートはできない
- SVGエクスポートには draw.io デスクトップ版（Electron アプリ）が必要
- パスの確認: `which drawio` が `/opt/drawio/drawio` でなければ、フルパスで `/opt/drawio/drawio` を使うこと

### 2. `shadow="1"` が原因でエクスポートが失敗する
- **症状**: `Error: Export failed: <ファイルパス>` と表示され、ログに `Exiting GPU process due to errors during initialization` が出る
- **原因**: WSL2 環境では GPU が利用できないため、shadow（ドロップシャドウ）の描画に失敗する
- **対処**: drawio ファイル内の `mxGraphModel` 要素で `shadow="1"` を `shadow="0"` に変更する

### 3. スタイル属性の構文エラー
- **症状**: エクスポートが失敗する、または描画が正しくない
- **原因**: drawio のスタイル文字列では `key=value;key=value;...` の形式が正しいが、`key:value` のようにコロンが混在している場合がある
- **対処**: `strokeColor:#FF6B9D` → `strokeColor=#FF6B9D` のように `:` を `=` に置換する

### 4. 依存関係のエラー
- **症状**: `dpkg -i` で draw.io インストール時に依存関係エラーが出る
- **対処**: `sudo apt --fix-broken install -y` を実行してから再度 `sudo apt-get install -y xvfb` を実行する

### 5. dbus エラーは無視してよい
- **症状**: `Failed to connect to the bus: Failed to connect to socket /run/user/1000/bus` が大量に出る
- **原因**: WSL2 に dbus サービスが起動していないため
- **影響**: エクスポート処理自体には影響しない。無視してよい

### 6. Windows 側の draw.io からのエクスポートは失敗する
- WSL2 から `/mnt/c/Program Files/draw.io/draw.io.exe` を呼び出してのエクスポートは `Export failed` となり動作しない
- WSL2 上の Linux 版 draw.io + xvfb を使うこと
