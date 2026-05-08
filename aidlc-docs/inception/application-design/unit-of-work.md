# ユニット定義

## ユニット一覧

| ID | ユニット名 | コンポーネント | 技術 | 優先度 |
|----|-----------|--------------|------|--------|
| U1 | インフラ | C7, C8, C9, C10, C12, C13 | CloudFormation (YAML) | 1 |
| U2 | フロントエンド | C1 | Quart + Jinja2 | 2 |
| U3 | エージェント | C2 | Strands Agents / AgentCore Runtime | 2 |
| U4 | 新規カテゴリ判定ツール | C11 | Lambda (Python) | 3 |
| U5 | モンスター生成ツール | C14 | Lambda (Python) | 3 |
| U6 | 進化判定ツール | C3 | Lambda (Python) | 4 |
| U7 | 技解放ツール | C4 | Lambda (Python) | 4 |
| U8 | 図鑑ツール | C6 | Lambda (Python) | 5 |
| U9 | モンスター画像生成 | C5 | Python CLI | 独立 |
| U10 | バトルシステム | (新規) | Lambda (Python) | 決勝 |

---

## ユニット詳細

### U1: インフラ
**責務**: 全共有リソースのCloudFormation定義・デプロイ

**含むリソース**:
- DynamoDB テーブル（Users, Monsters, DameActions, MonsterMaster, ImageMapping）
- S3 バケット（モンスター画像）
- Cognito ユーザープール
- Bedrock Guardrails 設定
- AgentCore Memory 設定
- AgentCore Runtime / Gateway 設定
- Lambda 実行ロール・ポリシー

---

### U2: フロントエンド
**責務**: 全画面のUI実装

**含む画面**:
- チャット画面（ダメ行動入力、深掘り対話、選択肢ボタン）
- ホーム画面（手持ちモンスター一覧、進捗表示）
- 図鑑画面（一覧、フィルタ、ヒント）
- モンスター詳細画面
- 認証画面（サインアップ/サインイン）
- 演出（孵化、進化、技解放、UR）
- ビジネスモード切替

---

### U3: エージェント
**責務**: C2のプロンプト設計・ツール定義・メモリ設定

**含む成果物**:
- システムプロンプト
- ツール定義（C3/C4/C6/C11/C14のMCPツール仕様）
- メモリ設定（パーソナライズ、行動パターン記憶）
- Guardrails連携設定

---

### U4: 新規カテゴリ判定ツール
**責務**: ユーザーのダメ行動カテゴリが新規か既存か判定

**メソッド**: `is_new_category()`

---

### U5: モンスター生成ツール
**責務**: 新モンスターの生成、レア度判定、画像選択

**メソッド**: `create_monster()`, `determine_rarity()`, `select_image()`

---

### U6: 進化判定ツール
**責務**: 進化条件チェック、分岐進化判定、進化適用

**メソッド**: `check_evolution()`, `determine_branch()`, `apply_evolution()`, `get_progress()`

---

### U7: 技解放ツール
**責務**: 技解放条件チェック、新技生成、技適用

**メソッド**: `check_skill_unlock()`, `generate_skill()`, `apply_skill()`

---

### U8: 図鑑ツール
**責務**: 図鑑データ取得、コンプリート率、ヒント、進化ツリー

**メソッド**: `get_collection()`, `get_completion_rate()`, `get_hints()`, `get_evolution_tree()`

---

### U9: モンスター画像生成
**責務**: モンスター画像の事前生成・S3アップロード・メタデータ登録

**実行方式**: Python CLIスクリプト（独立実行、デプロイフローとは別）

---

### U10: バトルシステム（決勝用）
**責務**: AI対戦、バトル報酬

**ステータス**: 定義のみ。MVP後に実装。

---

## コード構成戦略

```
damelog/
├── infra/                    # U1: CloudFormation
│   └── template.yaml
├── frontend/                 # U2: Quart + Jinja2
│   ├── app.py
│   ├── templates/
│   ├── static/
│   └── requirements.txt
├── agent/                    # U3: エージェント設定
│   ├── system_prompt.md
│   ├── tool_definitions/
│   └── memory_config.json
├── tools/                    # U4〜U8: Lambda ツール群
│   ├── category_check/       # U4: C11
│   │   ├── handler.py
│   │   └── requirements.txt
│   ├── monster_create/       # U5: C14
│   │   ├── handler.py
│   │   └── requirements.txt
│   ├── evolution/            # U6: C3
│   │   ├── handler.py
│   │   └── requirements.txt
│   ├── skill_unlock/         # U7: C4
│   │   ├── handler.py
│   │   └── requirements.txt
│   └── zukan/                # U8: C6
│       ├── handler.py
│       └── requirements.txt
├── image_gen/                # U9: 画像生成CLI
│   ├── generate.py
│   ├── prompts/
│   └── requirements.txt
└── aidlc-docs/               # ドキュメント
```

## 開発順序

1. **U1（インフラ）** → 全ユニットの基盤
2. **U9（画像生成）** → 独立実行、並行可能
3. **U2 + U3 + U4 + U5** → コアフロー縦串（ダメ行動→モンスター誕生）
4. **U6 + U7** → 進化・技解放フロー
5. **U8** → 図鑑フロー
6. **U10** → 決勝用バトル
