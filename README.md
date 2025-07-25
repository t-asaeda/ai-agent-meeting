# AIエージェント会議実験

## 概要
3大LLM（Gemini 2.5 Pro、Claude Opus 4、OpenAI o3 Pro）に同じ設計タスクを与えて、それぞれの個性と設計アプローチの違いを比較した実験プロジェクトです。

## 実験内容
- **タスク**: 基本的な四則演算ができる計算機プログラムの設計
- **形式**: AIエージェントによる仮想会議（PM、SA、LD、QAの4役職）
- **目的**: 各LLMの設計思想、表現方法、技術的アプローチの違いを観察

## ディレクトリ構成
```
ai-agent-meeting/
├── README.md           # このファイル
├── prompt/
│   └── AI_AGENT_MEETING_PROMPT.md  # 実験で使用したプロンプト
├── designs/            # 各モデルの設計ドキュメント
│   ├── gemini_2.5_pro.md
│   ├── claude_opus_4.md
│   └── openai_o3_pro.md
└── evaluation/         # 評価結果
    └── EVALUATION_DETAILED.md
```

## 実験結果の概要

### 各モデルの特徴

#### Gemini 2.5 Pro
- **設計スタイル**: 体系的・包括的
- **特徴**: サービス全体を見据えた設計、将来の拡張性を重視
- **強み**: 構造性と実用性のバランス

#### Claude Opus 4  
- **設計スタイル**: 詳細・実装指向
- **特徴**: 具体的なコード例とエラーハンドリングを重視
- **強み**: 実装詳細度と実用性

#### OpenAI o3 Pro
- **設計スタイル**: 簡潔・本質重視
- **特徴**: シンプルで理解しやすい設計
- **強み**: 革新性と実用性

### 評価マトリクス
各モデルを5つの観点で評価（◎:優秀、○:良好、△:改善の余地あり）

| 評価観点 | Gemini 2.5 Pro | Claude Opus 4 | OpenAI o3 Pro |
|---------|----------------|---------------|---------------|
| 構造性   | ◎              | ◎             | ○             |
| 実装詳細度| ○              | ○             | ○             |
| 実用性   | ◎              | △             | ◎             |
| 包括性   | △              | ◎             | △             |
| 革新性   | ○              | ○             | ◎             |

## 実験の意義
- LLMの「個性」を理解することで、適材適所の活用が可能に
- 各モデルの強みを活かした使い分けの指針を提供
- AI活用における選択基準の参考資料

## 関連リンク
- [ブログ記事](※公開後にURLを追加)
- [LT発表資料](※公開後にURLを追加)

## ライセンス
MIT License

## 著者
- 実験実施: t-asaeda
- 実施日: 2025年7月