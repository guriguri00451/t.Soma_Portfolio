# Units Generation — t.Soma Portfolio

**Version**: 1.0  
**Date**: 2026-06-17  
**Status**: Draft（承認待ち）

---

## 概要

Application Design で確定した構成を、実装可能な 5 ユニットに分割する。
各ユニットは前のユニットに依存する直列チェーンとし、U-01 の完成を前提に順番に実装する。

---

## ユニット一覧

| Unit | 名前 | 主な成果物 | 対応 FR / US |
|------|------|-----------|-------------|
| U-01 | Project Foundation | Next.js 基盤・設計システム・スクロール連動スカイ | — |
| U-02 | Hero Section | Hero コンポーネント・パララックス・SVG 装飾 | FR-01, US-01 |
| U-03 | Skills Section | スキルデータ・SkillBadge・Skills セクション | FR-02, US-02 |
| U-04 | Projects Section | MDX 処理・ProjectCard・ProjectModal | FR-03, US-03, US-05, US-08 |
| U-05 | Navigation & Footer | ナビゲーション・フッター・SNS リンク・統合 | FR-04, US-04, US-07 |

---

## 依存関係グラフ

```
U-01 Foundation
  └─ U-02 Hero
       └─ U-03 Skills
            └─ U-04 Projects
                 └─ U-05 Navigation & Footer
                      └─ Build & Test → GitHub Pages
```

---

## U-01: Project Foundation

### 目的

全セクションが乗る「地盤」を作る。ここが完成するまで他のユニットは開始できない。

### 成果物一覧

| ファイル / 設定 | 内容 |
|--------------|------|
| `package.json` | next 15, typescript, tailwindcss v4, framer-motion, simple-icons, next-mdx-remote, gray-matter, @tailwindcss/typography |
| `next.config.ts` | `output: 'export'`, `basePath`, `trailingSlash`, `images.unoptimized` |
| `src/app/layout.tsx` | メタデータ、Google Fonts（M PLUS Rounded 1c / Noto Sans JP / Inter / JetBrains Mono）、グローバル CSS |
| `src/app/globals.css` | CSS カスタムプロパティ（昼パレット + 夕焼けパレット全変数）、ベーススタイル |
| `src/app/page.tsx` | セクションを並べるルートページ（セクションは後のユニットで追加） |
| `src/lib/types.ts` | `SkillLevel`, `Skill`, `SkillCategory`, `Project`, `SocialLink` の型定義 |
| `src/lib/sky-controller.ts` | スクロール進捗 → lerp → CSS 変数書き換えロジック |
| `src/components/ui/AnimatedSection.tsx` | `useInView` + `prefers-reduced-motion` 対応ラッパー |
| `src/components/ui/CursorGlow.tsx` | デスクトップ限定カーソルグロー（media query で制御） |
| `src/components/ui/SectionTitle.tsx` | セクション見出し共通コンポーネント |
| `.github/workflows/deploy.yml` | GitHub Actions（npm ci → build → peaceiris/actions-gh-pages@v4） |
| `tsconfig.json` | パスエイリアス `@/*` 設定 |

### 技術詳細

**SkyController（`src/lib/sky-controller.ts`）**
```
scrollProgress = window.scrollY / (document.body.scrollHeight - window.innerHeight)
lerp(a, b, t) = a + (b - a) * t
各 CSS 変数 = lerp(昼の値, 夕焼けの値, scrollProgress)
```
- `useEffect` + `addEventListener('scroll', ...)` でリアルタイム更新
- `requestAnimationFrame` でスロットリング

**CSS カスタムプロパティ（`globals.css`）**
```css
:root {
  /* 昼パレット */
  --color-sky-light:   #EAF6FD;
  --color-sky:         #4BA3E3;
  --color-sky-deep:    #1B7EC4;
  --color-cloud:       #F7FBFF;
  --color-cloud-warm:  #FFE4E1;
  --color-silhouette:  #1C1C2E;
  --color-sun:         #FFD166;
  --color-text:        #2C3E50;
  --color-text-muted:  #7F8C9A;

  /* 夕焼けパレット（JS が lerp して上書き） */
  --color-sunset-sky:     #FF8C42;
  --color-sunset-pink:    #FF6B9D;
  --color-sunset-purple:  #7B4F9E;
  --color-sunset-horizon: #FFD166;
  --color-sunset-cloud:   #FFBC80;
}
```

### 完了基準

- [ ] `npm run build` がエラーなし
- [ ] `npm run dev` でローカル確認できる
- [ ] GitHub Actions が push 時に動作する（初回は空ページで OK）
- [ ] スクロールで背景色が昼 → 夕焼けに変化する（視認できれば OK）

---

## U-02: Hero Section

### 目的

ファーストビューの世界観を確立する。採用担当が開いた瞬間に「面白い」と感じるビジュアルを実装。

### 成果物一覧

| ファイル | 内容 |
|---------|------|
| `src/components/sections/Hero.tsx` | 名前・キャッチコピー・プロフィール画像・スクロール誘導アロー |
| `src/components/ui/ParallaxLayer.tsx` | 前景 / 中景 / 遠景の速度制御レイヤー |
| `src/components/ui/PowerLine.tsx` | 電線 SVG（セクション上部・Hero 前景） |
| `src/components/ui/CloudLayer.tsx` | 入道雲 SVG / 画像（中景・parallax 対応） |
| `src/components/ui/MeteorCanvas.tsx` | 遠景の流星・粒子アニメーション（CSS animation） |
| `src/components/ui/BirdSprite.tsx` | 電線の鳥・クリックで飛び立つ（Framer Motion） |
| `public/images/profile.jpg` | プレースホルダー（後から差し替え） |

### パララックス設計

| レイヤー | 要素 | 速度係数 | 実装 |
|---------|------|---------|------|
| 前景（Foreground） | 電線・シルエット | 1.2（速い） | `translateY(scrollY * 0.2)` |
| 中景（Midground） | 入道雲 | 0.4（遅い） | `translateY(scrollY * -0.15)` |
| 遠景（Background） | 空・流星 | 0（固定） | CSS 背景のみ |

### アニメーション詳細

| 対象 | トリガー | 実装 |
|------|---------|------|
| Hero テキスト群 | ページロード | Framer Motion stagger（0.0s → 0.15s → 0.3s → 0.45s） |
| プロフィール画像 | ページロード | scale 0.8 → 1.0、opacity 0 → 1（0.5s、easeOut） |
| スクロール誘導アロー | 常時 | CSS bounce アニメーション |
| 鳥 | クリック | opacity 0 + translateY(-80px) + rotate（Framer Motion、0.6s） |
| 流星 | 常時（ループ） | CSS `@keyframes` で右上 → 左下へ流れる、ランダム delay |

### 完了基準

- [ ] Hero がスクロールなしでファーストビューに収まる（US-01）
- [ ] パララックスが 3 レイヤーで動作する
- [ ] 鳥をクリックすると飛び立つ
- [ ] 流星がランダムに流れる
- [ ] モバイル（320px）でレイアウト崩れなし
- [ ] `prefers-reduced-motion` でアニメーション停止

---

## U-03: Skills Section

### 目的

技術スタックをカテゴリ別に、視覚的に魅力的な形で表示する。

### 成果物一覧

| ファイル | 内容 |
|---------|------|
| `src/data/skills.json` | 実際のスキルデータ（Soma の技術スタックを入力） |
| `src/components/ui/SkillBadge.tsx` | アイコン + 技術名 + レベル表示バッジ |
| `src/components/sections/Skills.tsx` | カテゴリ別グリッド表示 |
| `public/svgs/divider-powerline.svg` | セクション間電線区切り SVG |

### SkillBadge 設計

```
表示内容: [Simple Icons ロゴ] [技術名] [レベルラベル]
例: 🎯 TypeScript  得意😽

hover: wobble アニメーション（rotate -3deg → 3deg → 0、0.3s）
スクロール: AnimatedSection でカテゴリ内 stagger フェードイン
```

### レベル表示スタイル

| レベル | ラベル | 色 |
|-------|-------|----|
| `得意😽` | バッジ背景：`--color-sky` | アクセント強 |
| `人並み☺️` | バッジ背景：`--color-sky-light` | 標準 |
| `学習中💪` | バッジ背景：点線ボーダーのみ | 控えめ |

### 完了基準

- [ ] 最低 3 カテゴリ以上でスキルが分類されている（US-02）
- [ ] 各バッジに Simple Icons アイコンが表示される
- [ ] SkillBadge の wobble hover が動作する
- [ ] スクロール時に stagger フェードインが動作する
- [ ] セクション間電線 SVG が表示される
- [ ] スマートフォンで見やすく表示される（US-02）

---

## U-04: Projects Section

### 目的

制作物を魅力的に見せ、詳細を読んでもらえる構造を実装する。

### 成果物一覧

| ファイル | 内容 |
|---------|------|
| `src/lib/projects.ts` | MDX ファイル読み込み・フロントマターパース・`featured` ソート |
| `src/data/projects/project-sample.mdx` | サンプルプロジェクト MDX（2〜3 件） |
| `src/components/ui/ProjectCard.tsx` | サムネイル・タイトル・説明・技術タグ・リンクアイコン |
| `src/components/ui/ProjectModal.tsx` | MDX コンテンツ表示モーダル（ESC / 背景クリックで閉じる） |
| `src/components/sections/Projects.tsx` | featured 優先グリッド・モーダル状態管理 |

### ProjectCard hover 設計

```
通常: shadow-sm
hover:
  - translateY(-6px)
  - shadow-lg（空色グロー：box-shadow に --color-sky を使用）
  - transition: 0.25s cubic-bezier(0.34, 1.56, 0.64, 1)
```

### ProjectModal 設計

```
構造:
  - backdrop（全画面 overlay、blur + 半透明）
  - モーダルコンテナ（max-w-2xl、スクロール可能）
  - MDX レンダリングエリア
  - GitHub / Demo リンクボタン
  - 閉じるボタン（×）

クローズトリガー:
  - × ボタンクリック
  - backdrop クリック
  - Escape キー（useEffect でキーリスナー）

アニメーション（Framer Motion AnimatePresence）:
  backdrop: opacity 0 → 1
  modal: opacity 0 + translateY(40px) → opacity 1 + translateY(0)
```

### MDX ローダー（`src/lib/projects.ts`）

```typescript
// gray-matter でフロントマター解析
// next-mdx-remote/rsc で MDX → React へ変換
// featured: true のプロジェクトを先頭に並べ替え
// 静的ビルド対応（fs.readdir + fs.readFile）
```

### 完了基準

- [ ] ProjectCard にサムネイル・タイトル・説明・技術タグ・リンクが表示される（US-03）
- [ ] featured プロジェクトが先頭に表示される
- [ ] カードクリックでモーダルが開く
- [ ] ESC キー / 背景クリックでモーダルが閉じる
- [ ] スクロールロックが機能する（モーダル開放中 body overflow-hidden）
- [ ] MDX ファイル追加だけで新プロジェクトが増える（US-05）
- [ ] 外部リンクが `target="_blank"` で開く（US-03, US-08）
- [ ] `npm run build` が静的エクスポートで成功する

---

## U-05: Navigation & Footer

### 目的

全体をまとめる枠組みを完成させ、デプロイまで持っていく。

### 成果物一覧

| ファイル | 内容 |
|---------|------|
| `src/data/social-links.json` | GitHub / Zenn / X などのリンクデータ |
| `src/components/layout/Navigation.tsx` | スクロール連動ナビ + モバイルハンバーガーメニュー |
| `src/components/layout/Footer.tsx` | SNS リンク一覧 + 夕焼けシルエット装飾 |
| `src/app/page.tsx`（更新） | 全セクション + Navigation + Footer を統合、SkyController を接続 |

### Navigation 設計

```
スクロール量 < 50px:
  背景: 透明
  テキスト: --color-silhouette（暗）

スクロール量 >= 50px:
  背景: rgba(white, 0.8) + backdrop-blur
  テキスト: --color-silhouette
  border-bottom: 1px solid rgba(sky, 0.3)

アクティブセクション:
  IntersectionObserver で各セクション ID を監視
  対応リンクに下線 / カラーハイライト

モバイル:
  ハンバーガーアイコン（Framer Motion でメニューを slide-down）
  メニュー開放中は body scroll-lock
```

### Footer 設計

```
背景: 夕焼けグラデーション（--color-sunset-sky → --color-sunset-purple）
前景: 電線・鉄塔シルエット SVG
SNS リンク: アイコン + ラベル、新タブで開く
コピーライト: © 2026 Soma Toyota
```

### 完了基準

- [ ] ナビゲーションのアクティブセクション検知が機能する
- [ ] スクロールで背景 blur が適用される
- [ ] モバイルハンバーガーメニューが開閉する
- [ ] SNS リンクがアイコン付きで表示される（US-04）
- [ ] フッターに夕焼けシルエットが表示される
- [ ] ページ全体で SkyController が動作する（Hero 青空 → Footer 夕焼け）
- [ ] 320px モバイルで全セクション正常表示（US-07）
- [ ] Lighthouse Performance スコア 90 以上（US-01）
- [ ] `npm run build` + GitHub Pages デプロイ成功

---

## Construction フェーズ計画

| Unit | Functional Design | NFR | Code Generation |
|------|-----------------|-----|----------------|
| U-01 Foundation | スキップ（設定・型のみ） | スキップ | ✅ 実行 |
| U-02 Hero | ✅ 実行（SVG・パララックス設計） | スキップ | ✅ 実行 |
| U-03 Skills | スキップ（シンプル） | スキップ | ✅ 実行 |
| U-04 Projects | ✅ 実行（MDX ローダー・モーダル設計） | スキップ | ✅ 実行 |
| U-05 Nav & Footer | スキップ（シンプル） | スキップ | ✅ 実行 |
