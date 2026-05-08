# コンポーネントメソッド定義

## C1: Web フロントエンド

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `render_chat()` | - | HTML | チャット画面表示 |
| `render_home()` | user_id | HTML | ホーム画面（手持ちモンスター一覧+進捗） |
| `render_zukan()` | user_id, filter | HTML | 図鑑画面 |
| `render_monster_detail()` | monster_id | HTML | モンスター詳細画面 |
| `send_message()` | user_id, text | StreamingResponse | エージェントにメッセージ送信、ストリーミング受信 |
| `toggle_business_mode()` | mode | - | UI モード切り替え |
| `play_animation()` | event_type, data | - | 演出再生（孵化/進化/技解放/UR） |

---

## C2: ダメコレエージェント

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `process_message()` | user_id, text | StreamingResponse | ユーザー入力を処理し応答を生成 |
| `analyze_dame_action()` | text, context | {category, attributes, severity} | ダメ行動のカテゴリ・属性・深刻度を判定 |
| `generate_followup()` | action_analysis, history | text | 深掘り質問を生成 |
| `determine_event()` | user_id, action_result | {event_type, data} | 記録後のイベント判定（新モンスター/進化/技解放） |
| `generate_suggestions()` | user_id | [button_labels] | 過去履歴から選択肢ボタンを生成 |

---

## C3: 進化判定ツール

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `check_evolution()` | monster_id, action_count | {can_evolve, evolution_target, progress} | 進化条件チェック |
| `determine_branch()` | monster_id, action_history | evolution_target_id | 分岐進化先の決定 |
| `apply_evolution()` | monster_id, target_id | updated_monster | 進化を適用しモンスターデータ更新 |
| `get_progress()` | monster_id | {current, required, percentage} | 進化進捗取得 |

---

## C4: 技解放ツール

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `check_skill_unlock()` | monster_id, action_history | {can_unlock, condition_met} | 技解放条件チェック |
| `generate_skill()` | monster_id, condition_type | {skill_name, effect, description} | 新技を生成（LLM利用） |
| `apply_skill()` | monster_id, skill | updated_monster | 技をモンスターに追加 |

---

## C5: モンスター生成ツール

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `is_new_category()` | user_id, category | bool | 新規カテゴリ判定 |
| `create_monster()` | user_id, category, attributes, severity | new_monster | 新モンスター生成 |
| `determine_rarity()` | attributes, severity, conditions | rarity (N/R/SR/UR) | レア度判定 |
| `get_image_url()` | attributes, evolution_stage, rarity | image_url | 画像URLマッピング取得 |

---

## C6: 図鑑ツール

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `get_collection()` | user_id, filter | [monster_cards] | 図鑑一覧取得 |
| `get_completion_rate()` | user_id, filter | {collected, total, rate} | コンプリート率計算 |
| `get_hints()` | user_id | [hints] | 未獲得モンスターのヒント取得 |
| `get_evolution_tree()` | monster_master_id | tree_data | 進化ツリー取得 |
