# Workflow Plan — t.Soma Portfolio

**Version**: 1.0  
**Date**: 2026-06-17  
**Status**: Draft（承認待ち）

---

## フェーズ実行計画

| フェーズ | 実行 | 理由 |
|---------|------|------|
| Workspace Detection | ✅ 完了 | |
| Reverse Engineering | ⏭ スキップ | Greenfield |
| Requirements Analysis | ✅ 完了 | |
| User Stories | ✅ 完了 | |
| Workflow Planning | ✅ 現在 | |
| Application Design | ✅ 実行 | 新規コンポーネント設計が必要 |
| Units Generation | ✅ 実行 | 複数セクション = 複数ユニット |
| Construction (Per-Unit) | ✅ 実行 | ユニットごとに設計→コード生成 |
| Build and Test | ✅ 実行 | GitHub Pages デプロイ設定 |

---

## ユニット分割

### MVP ユニット（v1.0）

| Unit | 名前 | 内容 | 対応 FR / US |
|------|------|------|-------------|
| U-01 | Project Foundation | Next.js セットアップ、デプロイ基盤、レイアウト、デザインシステム | — |
| U-02 | Hero セクション | 自己紹介、アニメーション、プロフィール画像 | FR-01, US-01 |
| U-03 | Skills セクション | 技術スタック一覧、カテゴリ分類、視覚表現 | FR-02, US-02 |
| U-04 | Projects セクション | 実績カード、詳細モーダル、データ駆動設計 | FR-03, US-03, US-05 |
| U-05 | Links セクション | GitHub / SNS リンク、フッター | FR-04, US-04, US-07, US-08 |

### Phase 2 ユニット（v1.1 — 後から追加）

| Unit | 名前 | 内容 | 対応 FR / US |
|------|------|------|-------------|
| U-06 | Career セクション | 職歴・学歴タイムライン | FR-05, US-06 |
| U-07 | Blog セクション | 記事リンク一覧 | FR-06, US-06 |

---

## 実装順序（依存関係を考慮）

```
U-01 Project Foundation
  └─ U-02 Hero
       └─ U-03 Skills
            └─ U-04 Projects
                 └─ U-05 Links
                      └─ Build & Test → GitHub Pages デプロイ
```

U-01 が全ての基盤。各セクションは前のユニット完了後に実装する（共有コンポーネントへの依存のため）。

---

## 技術スタック確定

| カテゴリ | 選定 | 理由 |
|---------|------|------|
| フレームワーク | Next.js 15 (App Router) | SSG + 将来的な機能拡張 |
| 言語 | TypeScript | 型安全、保守性 |
| スタイリング | Tailwind CSS v4 | ユーティリティクラス、アニメーション対応 |
| アニメーション | Framer Motion | React ネイティブ、実装コスト低 |
| アイコン | Lucide React + Simple Icons | 技術ロゴは Simple Icons で網羅 |
| データ管理 | JSON（スキル・SNS）+ MDX（実績詳細） | ファイル追加だけで実績更新可能 |
| デプロイ | GitHub Actions → GitHub Pages | 無料、自動化 |
| パッケージ管理 | npm | シンプル、追加設定不要 |

---

## ワークフロー可視化

```
INCEPTION（完了）
├── Workspace Detection ✅
├── Requirements Analysis ✅
└── User Stories ✅

INCEPTION（続き）
├── Workflow Planning ← 現在
├── Application Design（次）
└── Units Generation

CONSTRUCTION
├── U-01: Foundation → Code Generation
├── U-02: Hero → Functional Design → Code Generation
├── U-03: Skills → Functional Design → Code Generation
├── U-04: Projects → Functional Design → Code Generation
├── U-05: Links → Code Generation
└── Build and Test → GitHub Pages Deploy
```

---

## 各ユニットの Construction サブフェーズ計画

| Unit | Functional Design | NFR Requirements | Code Generation |
|------|-----------------|-----------------|----------------|
| U-01 Foundation | スキップ（設定ファイル） | スキップ | ✅ 実行 |
| U-02 Hero | ✅ 実行（アニメーション設計） | スキップ | ✅ 実行 |
| U-03 Skills | ✅ 実行（データ構造定義） | スキップ | ✅ 実行 |
| U-04 Projects | ✅ 実行（MDX スキーマ・モーダル設計） | スキップ | ✅ 実行 |
| U-05 Links | スキップ（シンプル） | スキップ | ✅ 実行 |
