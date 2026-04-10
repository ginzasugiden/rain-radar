# 雨雲レーダーマップ 実装タスク

## プロジェクト概要

RainViewer API + Leaflet.js を使って、ウェザーニュース風の「雨雲アニメーションレーダー」を
GitHub Pages でホスティングできる単一HTMLファイルとして実装する。

---

## ディレクトリ構成

```
rain-radar/
├── index.html       ← メインファイル（全コードをここに集約）
└── README.md
```

---

## 機能要件

### 基本機能
- [x] Leaflet.js で日本地図を表示（初期表示は東京中心）
- [x] RainViewer API から雨雲レーダータイルを取得・表示
- [x] 過去2時間 ～ 現在 ～ 未来1時間（nowcast）のアニメーション再生
- [x] 再生 / 一時停止ボタン
- [x] 時刻スライダー（手動で時刻を選択可能）
- [x] 現在時刻の表示（例：「2時間前」「現在」「+30分予測」）
- [x] 現在地ボタン（Geolocation API でユーザー位置に移動）

### UI要件
- 黒背景ベースのダークテーマ（夜間の天気確認を想定）
- コントロールパネルは画面下部に固定
- モバイル対応（スマホでも使いやすいサイズ）
- 「過去」「現在」「予測」の区別が色でわかる時刻表示

---

## 技術仕様

### 使用ライブラリ（CDN）
```html
<!-- 地図 -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

### RainViewer API フロー

```javascript
// 1. タイムスタンプ一覧を取得
const res = await fetch('https://api.rainviewer.com/public/weather-maps.json');
const data = await res.json();

// data.radar.past   → 過去フレームのリスト
// data.radar.nowcast → 予測フレームのリスト
// 各フレーム: { time: unixTimestamp, path: "/v2/radar/..." }

// 2. タイルURLのフォーマット
// https://tilecache.rainviewer.com{path}/256/{z}/{x}/{y}/2/1_1.png
// ※ 最後の "2" はカラースキーム（2=レーダーカラー）
// ※ "1_1" はスムージング_スノー表示
```

### アニメーション制御

```javascript
// フレームは past + nowcast を結合して配列管理
// setInterval で 500ms ごとにフレームを切り替え
// 現在時刻に最も近いフレームを「現在」としてハイライト
```

---

## 実装詳細

### index.html の構成

```
<head>
  - meta viewport（モバイル対応）
  - Leaflet CSS CDN
  - カスタムCSS（インライン）
</head>
<body>
  - #map（全画面）
  - #control-panel（下部固定）
    - #time-display（現在選択中の時刻）
    - #slider（input[type=range]）
    - #btn-play（再生/停止）
    - #btn-locate（現在地）
    - #legend（カラー凡例）
  - Leaflet JS CDN
  - カスタムJS（インライン）
</body>
```

### カラー凡例

| 色 | 降水強度 |
|---|---|
| 水色 | 弱い雨（1mm/h未満） |
| 緑 | 小雨（1-5mm/h） |
| 黄 | 中雨（5-20mm/h） |
| 橙 | 強雨（20-50mm/h） |
| 赤 | 激しい雨（50mm/h以上） |

---

## デザイン仕様

- **テーマ**: ダーク / 気象・防災ツール風
- **背景色**: `#0a0e1a`（深い紺黒）
- **アクセント**: `#00d4ff`（シアン）
- **フォント**: `'Noto Sans JP', sans-serif`（日本語対応）
- **コントロールパネル**: 半透明黒 `rgba(10,14,26,0.85)` + blur backdrop
- **地図スタイル**: CartoDB Dark Matter（暗いベースマップ）
  - タイルURL: `https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png`

---

## スライダーの時刻ラベル表示ロジック

```javascript
function getTimeLabel(unixTime, referenceTime) {
  const diffMin = Math.round((unixTime - referenceTime) / 60);
  if (diffMin === 0) return '現在';
  if (diffMin > 0) return `+${diffMin}分 (予測)`;
  return `${diffMin}分前`;
}
```

---

## 完成後の確認事項

- [ ] ブラウザで `index.html` を直接開いてレーダーが表示されるか
- [ ] 再生ボタンでアニメーションが動くか
- [ ] スライダーで過去〜予測を手動で切り替えられるか
- [ ] スマホ（iPhone Safari / Android Chrome）で表示が崩れないか
- [ ] 現在地ボタンで地図が移動するか

---

## GitHub Pages デプロイ手順（実装完了後）

```bash
git init
git add .
git commit -m "feat: 雨雲レーダーマップ初期実装"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/rain-radar.git
git push -u origin main
# GitHub リポジトリの Settings > Pages > Branch: main / root で公開
```

---

## 注意事項

- RainViewer API は無料・APIキー不要（商用利用も可）
- タイルキャッシュは `tilecache.rainviewer.com` から直接取得
- CORS 制限なし（ブラウザから直接アクセス可能）
- ファイルを直接開く場合、Geolocation は HTTPS 環境（GitHub Pages）でないと動作しない場合あり
