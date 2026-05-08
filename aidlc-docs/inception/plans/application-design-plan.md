# アプリケーション設計プラン

## 実行チェックリスト
- [ ] Step 1: 質問への回答収集
- [ ] Step 2: コンポーネント定義（components.md）
- [ ] Step 3: コンポーネントメソッド定義（component-methods.md）
- [ ] Step 4: サービス定義（services.md）
- [ ] Step 5: コンポーネント依存関係（component-dependency.md）
- [ ] Step 6: 統合ドキュメント（application-design.md）

---

## 質問

## Question 1
コンポーネント分割の方針として、以下のどれが適切ですか？

A) レイヤー型（プレゼンテーション / ビジネスロジック / データアクセス）
B) 機能ドメイン型（記録ドメイン / モンスタードメイン / 図鑑ドメイン / 認証ドメイン）
C) エージェント中心型（エージェントが全ビジネスロジックを担い、ツールとして各機能を呼び出す）
X) Other (please describe after [Answer]: tag below)

[Answer]: C

**推薦理由**: Strands Agents + AgentCore を中心に据える設計。エージェントがオーケストレーターとなり、Gateway経由でツール（Lambda）を呼び出す構成は技術構成と一致。審査でもAgentCoreの活用度が示せる。

## Question 2
フロントエンド（Quart + Jinja2）とエージェントの通信方式は？

A) REST API（同期）— フロントエンドがAPIを叩き、レスポンスを待つ
B) WebSocket（双方向）— エージェントからのストリーミング応答をリアルタイム表示
C) Server-Sent Events（SSE）— エージェントの応答を逐次ストリーミング
X) Other (please describe after [Answer]: tag below)

[Answer]: X

REST + ストリーミングレスポンス（HTTP streaming）。AgentCore Runtime の InvokeAgentRuntime API がストリーミングレスポンス（contentStart→contentDelta→contentStop）を返すので、Quart の async generator でチャンクをそのままクライアントに流す。SSE（非推奨）ではなく HTTP chunked response。WebSocket ほど接続管理が複雑にならずMVP向き。

**理由**: AgentCore公式対応のストリーミングレスポンスを活用。チャット体験（文字が流れる表示）を実現しつつ、リクエストごとに完結するステートレス設計でシンプル。将来WebSocket移行時もエージェント側は変更不要。

## Question 3
モンスター画像（事前生成済み）の管理方式は？

A) S3に全画像を格納し、DynamoDBにメタデータ（属性×進化段階×レア度→画像URL）を持つ
B) S3に全画像を格納し、命名規則（例: `{属性}_{進化段階}_{レア度}.png`）で直接参照
C) CloudFront + S3 で配信し、DynamoDBでマッピング管理
X) Other (please describe after [Answer]: tag below)

[Answer]: A
