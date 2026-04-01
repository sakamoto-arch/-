---
name: competitive-intel
description: |
  競合・市場の投稿を蓄積して発信のハレーションリスクを防ぐインテリジェンスKBを構築・運用するスキル。
  「競合KBに追加して」「市場の声を蓄積したい」「発信前にハレーションチェックしたい」
  「競合投稿を分析して」「SNS発信のリスクを診断して」と言われたら必ずこのスキルを使う。
  ユーザーがXのURLや投稿テキストを貼ってきた場合も、このスキルで競合インテリジェンスとして分析・蓄積する。
metadata:
  author: custom
  version: "2.0.0"
  platform: claude-code
---

# 競合インテリジェンス KB（Claude Code版）

## ストレージ設定

KBデータは以下のJSONファイルで管理する：

```
~/.agents/skills/competitive-intel/references/kb.json
```

ファイルが存在しない場合は空配列として扱い、初回書き込み時に自動作成する。

---

## 操作の判定

| 発言パターン | 操作 |
|---|---|
| 「KBに追加して」「この投稿を蓄積して」「競合投稿を貼る」 | 追加 |
| 「KBを見せて」「蓄積を確認」「一覧を出して」 | 一覧 |
| 「ハレーションチェック」「発信前チェック」「この投稿は大丈夫？」 | チェック |

---

## 操作1：追加

### Step 1：投稿テキストを確認する

ユーザーが投稿テキストを渡していない場合は「分析したい投稿テキストを貼ってください」と聞く。

### Step 2：分析する

以下の項目を分析する（ユーザーには見せない）：

- claim：主張を1文で（日本語）
- stance：好戦的 / 懐疑的 / 支持 / 中立 のいずれか
- intensity：1〜5の整数（5が最も強い）
- topics：タグの配列
- fieldGap：現場認識との乖離を一言（日本語）
- summary：要点を2〜3行（日本語）

### Step 3：kb.jsonを読み込む

```bash
cat ~/.agents/skills/competitive-intel/references/kb.json 2>/dev/null || echo "[]"
```

### Step 4：新エントリを追加して保存する

既存配列の先頭に以下の形式で新エントリを追加し、Writeツールで上書き保存する：

```json
{
  "id": 1234567890,
  "date": "YYYY-MM-DD",
  "url": "XのURL（ある場合、なければ空文字）",
  "text": "投稿テキスト原文",
  "analysis": {
    "claim": "主張",
    "stance": "好戦的",
    "intensity": 4,
    "topics": ["LLMO", "SEO"],
    "fieldGap": "現場では...",
    "summary": "要点..."
  }
}
```

### Step 5：完了を報告する

```
✅ KB に追加しました（合計 N 件）

[claim]
スタンス: [stance] / 強度: [intensity]/5
現場乖離: [fieldGap]
```

---

## 操作2：一覧

### Step 1：kb.jsonを読み込む

```bash
cat ~/.agents/skills/competitive-intel/references/kb.json 2>/dev/null || echo "[]"
```

### Step 2：整形して表示する

```
📚 競合インテリジェンス KB（N件）

[1] YYYY-MM-DD | [stance] | 強度[intensity]/5
    [claim]
    乖離: [fieldGap]
    タグ: #tag1 #tag2
```

件数が0の場合は「まだ蓄積されていません」と伝える。

---

## 操作3：ハレーションチェック

### Step 1：ドラフトを確認する

渡されていない場合は「チェックしたいドラフトを貼ってください」と聞く。

### Step 2：kb.jsonを読み込む

```bash
cat ~/.agents/skills/competitive-intel/references/kb.json 2>/dev/null || echo "[]"
```

KBが空の場合はweb_searchで当該テーマの批判的意見を検索して代替監査を行う。

### Step 3：診断結果を出力する

```
🔍 ハレーションチェック結果

リスクレベル：高 / 中 / 低

[総評2〜3行]

想定される反論：
・[KBの claim から引用]（[date]の[stance]投稿より）

改善提案：
・[提案]
```

---

## その他のコマンド

- 「KBをリセットして」→ 確認後に空配列で上書き
- 「KBをエクスポートして」→ Markdown形式でチャットに出力
- 「〇〇トピックだけ見せて」→ topics でフィルタして一覧表示
