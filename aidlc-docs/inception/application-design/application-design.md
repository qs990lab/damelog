# アプリケーション設計（統合）

## アーキテクチャ概要

**方針**: エージェント中心型アーキテクチャ

Strands Agents（AgentCore Runtime上）がオーケストレーターとして全ビジネスロジックを制御し、AgentCore Gateway経由でツール（Lambda）を呼び出す。

**スケーラビリティ設計**:
- Lambda個別デプロイにより各ツールが独立スケール
- DynamoDBオンデマンドモードでキャパシティ自動調整
- AgentCore Runtimeがエージェント実行のスケーリングを管理
- S3 + CloudFront（将来）で画像配信を分離

## コンポーネント一覧

| ID | コンポーネント | 技術 | 責務 |
|----|--------------|------|------|
| C1 | Webフロントエンド | Quart + Jinja2 | UI表示、演出、モード切替 |
| C2 | ダメコレエージェント | Strands Agents / AgentCore Runtime | 対話、分析、オーケストレーション |
| C3 | 進化判定ツール | Lambda / Gateway | 進化条件チェック、分岐判定 |
| C4 | 技解放ツール | Lambda / Gateway | 技解放条件チェック、技生成 |
| C5 | モンスター画像生成ツール | Python CLI | モンスター画像の事前生成 |
| C6 | 図鑑ツール | Lambda / Gateway | 図鑑取得、コンプリート率、ヒント |
| C7 | データストア | DynamoDB | 全データ永続化 |
| C8 | 画像ストレージ | S3 | 事前生成画像の格納・配信 |
| C9 | 認証 | Cognito | サインアップ/サインイン |
| C10 | コンテンツフィルタ | Bedrock Guardrails | 入力安全性チェック |
| C11 | 新規カテゴリ判定ツール | Lambda / Gateway | 新規カテゴリ判定 |
| C12 | メモリ | AgentCore Memory | 対話コンテキスト、パーソナライズ |
| C13 | LLM | Bedrock (Claude) | テキスト生成、分析、判定 |
| C14 | モンスター生成ツール | Lambda / Gateway | モンスター生成、レア度判定、画像選択 |

## 通信方式

| 経路 | プロトコル |
|------|-----------|
| ユーザー → C1 | HTTPS |
| C1 → C2 | HTTP streaming（InvokeAgentRuntime） |
| C2 → C3/C4/C6/C11/C14 | AgentCore Gateway（MCP / Streamable HTTP） |
| C2 → C10 | Bedrock Guardrails API |
| C2 → C12 | AgentCore Memory API |
| C2 → C13 | Bedrock InvokeModel API |
| C3/C4/C6/C11/C14 → C7 | DynamoDB SDK |
| C1 → C8 | S3 URL 直接参照 |
| C1 → C9 | Cognito SDK |

## 主要フロー

1. **モンスター画像生成**: C5で事前生成→C8に格納
2. **ダメ行動記録**: C1→C2→(Guardrails)→分析→C11で判定→(新規)C14で生成 or (既存)C3/C4で進化・技解放→C1に演出イベント
3. **図鑑閲覧**: C1→C2→C6→C1
4. **認証**: C1→C9→JWT→以降全リクエストに付与

## 詳細ドキュメント

- [コンポーネント定義](components.md)
- [コンポーネントメソッド](component-methods.md)
- [サービス定義](services.md)
- [コンポーネント依存関係](component-dependency.md)
