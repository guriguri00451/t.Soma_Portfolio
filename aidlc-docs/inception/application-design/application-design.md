# Application Design — t.Soma Portfolio

**Version**: 1.0  
**Date**: 2026-06-17  
**Status**: Draft（承認待ち）

---

## 1. ページ構成・ルーティング

シングルページ構成（1 ページスクロール）。セクション間はアンカーリンクでナビゲートする。

```
/          ← メインページ（全セクション）
/projects/ ← （将来的に個別プロジェクトページ対応）
```

---

## 2. ディレクトリ構成

```
src/
├── app/
│   ├── layout.tsx          # メタデータ、グローバルスタイル
│   ├── page.tsx            # 全セクションを並べるルートページ
│   └── globals.css
│
├── components/
│   ├── layout/
│   │   ├── Navigation.tsx  # ヘッダーナビ（スクロール連動）
│   │   └── Footer.tsx      # SNS リンク
│   ├── sections/
│   │   ├── Hero.tsx        # 自己紹介セクション
│   │   ├── Skills.tsx      # スキルセクション
│   │   └── Projects.tsx    # 実績セクション
│   └── ui/
│       ├── ProjectCard.tsx  # 実績カード
│       ├── ProjectModal.tsx # 実績詳細モーダル
│       ├── SkillBadge.tsx   # スキルバッジ
│       ├── SectionTitle.tsx # セクション見出し（共通）
│       └── AnimatedSection.tsx # スクロールアニメーションラッパー
│
├── data/
│   ├── skills.json         # スキルデータ
│   ├── social-links.json   # SNS リンクデータ
│   └── projects/
│       ├── project-a.mdx   # プロジェクト詳細（MDX）
│       └── project-b.mdx
│
├── lib/
│   ├── projects.ts         # MDX 読み込み・パース処理
│   └── types.ts            # 型定義（全体で共有）
│
└── public/
    └── images/
        └── projects/       # プロジェクトサムネイル
```

---

## 3. TypeScript 型定義

```typescript
// lib/types.ts

/** スキルの習熟度 */
type SkillLevel = '学習中💪' | '人並み☺️' | '得意😽'

/** 個別スキル */
type Skill = {
  name: string       // 表示名（例: "TypeScript"）
  icon: string       // Simple Icons のスラッグ（例: "typescript"）
  level: SkillLevel
}

/** スキルカテゴリ */
type SkillCategory = {
  id: string         // ユニーク ID（例: "languages"）
  label: string      // 表示名（例: "言語"）
  skills: Skill[]
}

/** プロジェクト（MDX フロントマター + コンテンツ） */
type Project = {
  slug: string          // ファイル名から自動生成（例: "game-a"）
  title: string
  description: string   // 一覧カード用の短い説明
  thumbnail: string     // /images/projects/xxx.png
  techs: string[]       // 使用技術タグ（例: ["Unity", "C#", "Godot"]）
  githubUrl?: string    // リポジトリ URL（非公開の場合は省略）
  demoUrl?: string      // デモ / プレイ URL（任意）
  featured: boolean     // トップに表示するか
  date: string          // 完成時期（例: "2024-06"）
  content: string       // MDX 本文（制作背景・工夫した点など）
}

/** SNS / 外部リンク */
type SocialLink = {
  name: string     // 表示名（例: "GitHub"）
  url: string      // プロフィール URL
  icon: string     // Simple Icons スラッグ（例: "github"）
}
```

---

## 4. データスキーマ

### 4.1 skills.json

```json
{
  "categories": [
    {
      "id": "languages",
      "label": "言語",
      "skills": [
        { "name": "TypeScript", "icon": "typescript", "level": "得意😽" },
        { "name": "C#", "icon": "csharp", "level": "人並み☺️" }
      ]
    },
    {
      "id": "frameworks",
      "label": "フレームワーク",
      "skills": [
        { "name": "Next.js", "icon": "nextdotjs", "level": "人並み☺️" },
        { "name": "React", "icon": "react", "level": "人並み☺️" }
      ]
    },
    {
      "id": "game",
      "label": "ゲーム開発",
      "skills": [
        { "name": "Unity", "icon": "unity", "level": "人並み☺️" },
        { "name": "Godot", "icon": "godotengine", "level": "学習中💪" }
      ]
    }
  ]
}
```

### 4.2 プロジェクト MDX（projects/xxx.mdx）

```mdx
---
title: "プロジェクト名"
description: "カード表示用の一行説明（80字以内）"
thumbnail: "/images/projects/project-a.png"
techs: ["Unity", "C#"]
githubUrl: "https://github.com/..."
demoUrl: ""
featured: true
date: "2024-06"
---

## 制作背景

このプロジェクトを作った動機...

## 工夫した点

技術的にチャレンジしたこと...
```

### 4.3 social-links.json

```json
[
  { "name": "GitHub", "url": "https://github.com/...", "icon": "github" },
  { "name": "Zenn",   "url": "https://zenn.dev/...",  "icon": "zenn" },
  { "name": "X",      "url": "https://x.com/...",     "icon": "x" }
]
```

---

## 5. コンポーネント設計

### Navigation

```
props: なし（グローバルコンポーネント）
状態:  activeSection（スクロール位置から自動判定）
動作:
  - スクロール量に応じて背景を半透明化（blur + backdrop-filter）
  - アクティブセクションのリンクをハイライト
  - モバイルではハンバーガーメニュー
```

### Hero

```
props: なし（静的コンテンツ）
内容:
  - 名前（大きめのテキスト）
  - キャッチコピー（1〜2行）
  - プロフィール画像
  - スクロール誘導アロー
アニメーション:
  - ページロード時に上からフェードイン（stagger）
  - プロフィール画像はスケールイン
```

### Skills

```
props: categories: SkillCategory[]
内容:
  - カテゴリタブ or セクション分割
  - SkillBadge を Grid 表示
アニメーション:
  - スクロールで表示時にフェードイン（AnimatedSection 使用）
```

### Projects

```
props: projects: Project[]
内容:
  - featured プロジェクトを上部に表示
  - ProjectCard のグリッド
  - カードクリックで ProjectModal を開く
```

### ProjectCard

```
props: project: Project, onOpen: (project: Project) => void
内容:
  - サムネイル画像
  - タイトル・説明
  - 技術タグ（techs）
  - GitHub / Demo リンクアイコン
アニメーション:
  - hover で軽いリフトアップ（translateY + shadow）
```

### ProjectModal

```
props: project: Project | null, onClose: () => void
動作:
  - Escape キー / 背景クリックで閉じる
  - MDX コンテンツを表示
  - スクロールロック（body overflow hidden）
アニメーション:
  - Framer Motion AnimatePresence でフェードイン/アウト
```

### AnimatedSection

```
props: children: ReactNode, delay?: number
動作:
  - Framer Motion useInView でビューポート内に入ったら
    フェードイン + 上方向スライドイン
  - prefers-reduced-motion を尊重（アニメーション無効化）
```

---

## 6. アニメーション設計

### 基本方針
- すべてのアニメーションは `prefers-reduced-motion: reduce` で無効化
- ページロード → セクションスクロール → カードインタラクションの 3 レイヤー

### アニメーション一覧

| 対象 | トリガー | 内容 |
|------|---------|------|
| Hero テキスト | ページロード | stagger フェードイン（上→下、0.2s ずつ） |
| Hero 画像 | ページロード | スケールイン（0.8 → 1.0） |
| 各セクション | スクロール | `useInView` フェードイン + translateY(-20px → 0) |
| ProjectCard | hover | translateY(-4px) + shadow 強調 |
| ProjectModal | open/close | `AnimatePresence` backdrop フェード + コンテンツスライドイン |
| SkillBadge | スクロール | stagger フェードイン（カテゴリ内で順番に） |

---

## 7. GitHub Actions デプロイ設定（設計）

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./out
```

---

## 8. グラフィックデザイン方針

### テーマ

**「夏の青空と電線」×「ノスタルジック・フューチャー」**
シンプルな仕組みだが、触っていて面白いサイトを目指す。
日本の夏の原風景（巨大な入道雲・青空・電線）をモチーフに、広大な空への憧れやスケール感と、日常の延長にある懐かしさを両立する。アニメイラスト調のクリーンでエモーショナルな世界観を構築する。

---

### カラーパレット

画像の持つ「青のグラデーション」と「雲に当たる温かい光」、そして「シルエット」の要素を統合。

| 役割 | 変数名 | カラーコード | 用途 |
|------|--------|------------|------|
| 背景（空・薄） | `--color-sky-light` | `#EAF6FD` | ページ背景の下部、雲の隙間 |
| メイン（空） | `--color-sky` | `#4BA3E3` | 空の中間層、アクセント、ボタン |
| 空（深） | `--color-sky-deep` | `#1B7EC4` | 画面上部の深い空、ホバー、強調 |
| 雲（白） | `--color-cloud` | `#F7FBFF` | メインの雲、カード背景、コンテナ |
| 雲（ハイライト） | `--color-cloud-warm` | `#FFE4E1` | 雲の縁など、太陽光が当たる淡いピンク/ピーチ |
| 電線・シルエット | `--color-silhouette` | `#1C1C2E` | 前景インフラ・人物シルエット、テキスト、電線装飾 |
| 太陽（金） | `--color-sun` | `#FFD166` | CTA、流星、ポイントアクセント |
| テキスト（本文） | `--color-text` | `#2C3E50` | 本文テキスト |
| テキスト（弱） | `--color-text-muted` | `#7F8C9A` | サブテキスト、キャプション |

---

### 空のカラーパターン（昼 → 夕焼け）

スクロール進捗に連動して、空が昼から夕焼けへ **グラデーション遷移** する。
CSS カスタムプロパティを `scrollProgress`（0.0〜1.0）で補間し `lerp` で滑らかに切り替える。

#### 夕焼けパレット

| 役割 | 変数名 | カラーコード | 用途 |
|------|--------|------------|------|
| 空（夕焼け上部） | `--color-sunset-sky` | `#FF8C42` | 夕焼け空の上部オレンジ |
| 空（夕焼け中央） | `--color-sunset-pink` | `#FF6B9D` | ピンク〜マゼンタ帯 |
| 空（夕焼け下部） | `--color-sunset-purple` | `#7B4F9E` | 地平線付近の薄紫 |
| 地平線の光 | `--color-sunset-horizon` | `#FFD166` | 太陽色と共通、地平線の金 |
| 雲（夕焼け） | `--color-sunset-cloud` | `#FFBC80` | 茜色に染まった雲 |

#### スクロール連動のしくみ

```
scrollProgress = 0.0  → 昼パレット（青空）
scrollProgress = 0.5  → 中間（夕方直前、やや橙がかった空）
scrollProgress = 1.0  → 夕焼けパレット（フッター付近）
```

- `window.scrollY / (document.body.scrollHeight - window.innerHeight)` で進捗を計算
- CSS 変数を JS から直接書き換え（`document.documentElement.style.setProperty`）
- ページ全体の背景グラデーションが連動して変化
- シルエット（電線・建物）は終始 `--color-silhouette` で固定（昼も夕も黒）

#### セクション対応イメージ

| セクション | 空の状態 |
|-----------|---------|
| Hero | 青空（昼）|
| Skills | 昼〜夕方の移行 |
| Projects | 夕焼け前（橙がかった空） |
| Footer | 完全な夕焼け + 電線・鉄塔シルエット |

---

### フォント

| 役割 | フォント | 備考 |
|------|---------|------|
| 日本語 見出し | **M PLUS Rounded 1c** | 丸みがあり夏らしい柔らかさ |
| 日本語 本文 | **Noto Sans JP** | 読みやすく汎用性が高い。細めのウェイトでクリーンに |
| 英数字 見出し | **Inter** | クリーンでモダン |
| コード | **JetBrains Mono** | スキルタグ・技術名に使用 |

Google Fonts から読み込み（`next/font/google` 経由）。

---

### モチーフの使い方（空・雲・シルエット）

広大なレイヤー構造（前景・中景・遠景）を意識して要素を配置する。

| 場所 | 使い方 |
|------|-------|
| ヒーロー背景 | 下部に鉄塔・船などのシルエット（前景）。中央〜上部へ巨大な入道雲が広がる構図 |
| セクション区切り | SVG の電線（水平線）で各セクションを分割し、日常感を演出 |
| コンテンツエリア | 雲のレイヤーに重なるようにコンテンツカードを配置 |
| 装飾要素 | 遠景の空に微細な流星や風に舞う粒子を配置 |
| フッター | 電線・鉄塔のシルエット + 淡い夕焼け（ピンク〜オレンジ）グラデーションで締める |

---

### インタラクション設計（触り心地）

| インタラクション | 内容 | 狙い |
|----------------|------|------|
| 電線の鳥クリック | 鳥が飛び立つ（Lottie or CSS アニメーション） | 思わず触りたくなる仕掛け |
| スクロール視差（Parallax） | 前景（電線・シルエット）は早く・中景（巨大な雲）は遅く・遠景（空・流星）は動かない | 圧倒的な奥行きとスケール感の演出 |
| ProjectCard hover | カードが浮き上がり、背景に影と空色グロー | 気持ちよい手触り感 |
| 空間装飾のアニメーション | 画面内をゆっくりと流星が落ちる、または粒子が風に舞う | 神秘性とノスタルジックな雰囲気の強調 |
| SkillBadge hover | バッジが軽く揺れる（wobble アニメーション） | 遊び心 |
| カーソル | デスクトップのみ：カーソル周辺に薄いグロー | サイト全体の没入感向上 |

---

### ライト / ダークモード
v1 はライトモードのみ。空の色調がテーマの核なのでダークモードは別テーマとして v2 以降で検討。

---

## 9. 未解決事項（実装時に決定）

- [ ] プロフィール画像のスタイル（円形 vs 正方形 + 角丸）
- [ ] Navigation のモバイルメニューアニメーション詳細
- [ ] 電線 SVG の詳細デザイン（本数・太さ・カーブ具合）
- [ ] 鳥アニメーションの実装方法（Lottie vs CSS vs Framer Motion）
