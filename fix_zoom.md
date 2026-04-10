# 修正タスク：RainViewer ズームレベルエラー対応

## 問題

ズームインすると地図タイルに「Zoom Level Not Supported」と表示される。

## 原因

RainViewer のレーダータイルは **ズームレベル 1〜12** までしかサポートしていない。
Leaflet の地図（CartoDB）はズーム14〜18まで拡大できるが、
RainViewer レイヤーはズーム13以上でタイルが存在しないためエラー表示になる。

## 修正内容

### 1. Leaflet 地図の最大ズームを制限

`index.html` 内の Leaflet マップ初期化部分を以下のように修正：

```javascript
// 修正前（例）
const map = L.map('map', {
  center: [35.6762, 139.6503],
  zoom: 7
});

// 修正後
const map = L.map('map', {
  center: [35.6762, 139.6503],
  zoom: 7,
  maxZoom: 12,   // ← RainViewer のサポート上限に合わせる
  minZoom: 3
});
```

### 2. RainViewer タイルレイヤーに maxZoom を明示

```javascript
// RainViewer タイルレイヤー生成箇所に maxZoom: 12 を追加
L.tileLayer(
  `https://tilecache.rainviewer.com${frame.path}/256/{z}/{x}/{y}/2/1_1.png`,
  {
    opacity: 0.6,
    maxZoom: 12,        // ← 追加
    maxNativeZoom: 12,  // ← 追加（これが重要）
    tileSize: 256,
    zIndex: 10
  }
);
```

### 3. ベースマップ（CartoDB）は maxNativeZoom を維持

ベースマップのタイルレイヤーは変更不要。
CartoDB はズーム18まで対応しているが、地図全体の maxZoom を 12 に制限することで
ユーザーがそれ以上ズームできなくなる。

## 修正後の期待動作

- ズームレベル 3〜12 の範囲でのみ操作可能
- 「Zoom Level Not Supported」エラーが完全に消える
- レーダーオーバーレイが全ズームレベルで正常表示される

## 補足

もし「もう少し細かく見たい」場合は以下の代替案を検討：
- 気象庁の降水ナウキャストタイル（最大ズーム10）に切り替える
- RainViewer の高解像度タイル（`/512/` エンドポイント）を試す（最大ズーム13対応の場合あり）
