# Scenario Architect

シナリオをフローチャート形式で視覚的に管理するためのブラウザベースツールです。  
`index.html` をブラウザで開くだけで動作します（サーバー不要 / スマートフォン対応）。
https://sashimihoutyou.github.io/ScenarioManager/
---

## ノードの種類

| 種別 | 色 | 用途 |
|------|------|------|
| **TEXT** | 青 | セリフ・地の文・演出など通常のシーン |
| **CHOICE** | 赤 | 分岐ポイント。子 OPTION ノードを自動生成 |
| **OPTION** | 橙 | CHOICE の各分岐先ラベル |
| **SPRITE** | 紫 | 立ち絵キャラクターの登場/退場/表情切り替え |

---

## 基本操作

### ノードの作成
上部タブバーの **＋ テキスト** / **＋ 選択肢** / **＋ 立ち絵** から追加します。

### ノードの編集
タップすると下部パネルが開きます。

- **TEXT / CHOICE**: テキストを入力して「完了」
- **SPRITE**: キャラ名・立ち位置・表情・アクション（登場/退場）を設定

### ノードの接続（方向ルール）
1. 接続元をタップ（ハイライト）
2. 接続先をタップ → 矢印が引かれる

接続は **常に左→右（x 座標が小さい側 → 大きい側）** に統一されます。  
どちらを先にタップしても向きは自動的に決まります。同じ組み合わせを再タップすると解除。

### ノードの移動
ドラッグで自由移動。CHOICE を移動すると配下の OPTION が追従します。

### コンテキストメニュー（長押し）

| 操作 | 説明 |
|------|------|
| ロック切替 | 固定/解除。固定ノードはタップしても編集パネルが開かない |
| コピー | 複製（OPTION は対象外）|
| 削除 | ノードと関連エッジを削除。CHOICE は配下 OPTION も一括削除 |

---

## タブバー（左右スワイプで全ボタンにアクセス）

| ボタン | 機能 |
|--------|------|
| ＋ テキスト | TEXT ノードを追加 |
| ＋ 選択肢 | CHOICE ノードを追加（デフォルト 2 分岐）|
| ＋ 立ち絵 | SPRITE ノードを追加 |
| 💾 保存 | ツール用 JSON をダウンロード |
| 📂 読込 | 保存した JSON を読み込む |
| 🖼 画像 | 現在の表示を PNG でダウンロード |
| 📤 Dialogic出力 | Dialogic 2.x 用 JSON をエクスポート（後述）|
| 🎯 全体表示 | 全ノードが収まるようビューを自動調整 |
| ↩ 元に戻す | 直前の操作を取り消す（最大 50 段階）|
| ↪ やり直す | 元に戻した操作を再適用（最大 50 段階）|

---

## ミニマップ（右上）

ドラッグ / ズーム / 全体表示 / ミニマップタップ中と、その終了後 1 秒間のみ表示されます。

- 白い矩形 = 現在の表示範囲（カメラ移動中もリアルタイム追従）
- タップするとその位置にカメラがジャンプ
- 他の場所をタップすると即座に非表示

スクロール可能な仮想空間は ±4000 × ±3000 に制限されています。

---

## キーボードショートカット

| キー | 機能 |
|------|------|
| `Delete` / `Backspace` | 選択中のノードを削除 |
| `Ctrl + Z` / `⌘ + Z` | 元に戻す |
| `Ctrl + Y` / `⌘ + Y` | やり直す |
| `Ctrl + Shift + Z` | やり直す（代替）|

---

## Dialogic 2.x へのエクスポート

### 設計方針

Dialogic 2 の `.dtl` / `.dch` ファイルは公式スキーマが公開されておらず、マイナーバージョンで変更されることがあります。  
そのため、このツールでは **JSON 交換フォーマット** を出力し、Godot 側の **GDScript インポーター** が読み込む方式を採用しています。

- **ツール側**: `dialogic_YYYY-MM-DD.json` を出力するだけ
- **Godot 側**: 以下の GDScript を一度プロジェクトに追加
- **メリット**: Dialogic のバージョンが上がっても GDScript 側だけ修正すれば済む

### ワークフロー

```
スマホ/PC で index.html を開く
    ↓
シナリオを編集
    ↓
「📤 Dialogic出力」ボタン → dialogic_YYYY-MM-DD.json をダウンロード
    ↓
Godot プロジェクトの res://dialogic/imports/ にコピー
    ↓
Godot エディタ内でインポーターを実行（またはゲーム起動時に自動読込）
```

### エクスポート JSON フォーマット

```json
{
  "_format": "scenario_architect_dialogic_v1",
  "_note":   "GDScript インポーターと組み合わせて使用してください。",
  "timelines": [
    {
      "id": "main",
      "events": [
        {
          "event_type": "dialogic_character",
          "character":  "Alice",
          "portrait":   "smile",
          "position":   "left",
          "action":     "join"
        },
        {
          "event_type": "dialogic_text",
          "text":       "こんにちは！"
        },
        {
          "event_type": "dialogic_choice",
          "question":   "どうする？",
          "options": [
            { "text": "はい",   "timeline": "timeline_opt_xxx_1" },
            { "text": "いいえ", "timeline": "timeline_opt_xxx_2" }
          ]
        }
      ]
    },
    {
      "id": "timeline_opt_xxx_1",
      "events": [
        { "event_type": "dialogic_text", "text": "はいを選びました" }
      ]
    }
  ]
}
```

#### event_type の種類

| event_type | フィールド | Dialogic の対応イベント |
|---|---|---|
| `dialogic_text` | `text` | Text Event |
| `dialogic_character` | `character`, `portrait`, `position` (left/right), `action` (join/leave) | Character Event |
| `dialogic_choice` | `question`, `options[]` | Choice Event |
| `dialogic_jump` | `timeline` | Jump Event（ループ・共通合流点） |

### GDScript インポーターのサンプル

以下を `res://addons/scenario_importer/import.gd` に配置し、エディタプラグインとして有効化してください。

```gdscript
# scenario_importer/import.gd
# Dialogic 2.x 向け JSON インポーター
# 使用方法: ScenarioImporter.import_file("res://dialogic/imports/dialogic_YYYY-MM-DD.json")
class_name ScenarioImporter

const DIALOGIC_TIMELINE_PATH = "res://dialogic/timelines/"
const DIALOGIC_CHARACTER_PATH = "res://dialogic/characters/"

static func import_file(json_path: String) -> void:
    var file = FileAccess.open(json_path, FileAccess.READ)
    if not file:
        push_error("ScenarioImporter: ファイルを開けません: " + json_path)
        return

    var json = JSON.new()
    var err  = json.parse(file.get_as_text())
    file.close()
    if err != OK:
        push_error("ScenarioImporter: JSON パースエラー")
        return

    var data = json.get_data()
    if data.get("_format") != "scenario_architect_dialogic_v1":
        push_error("ScenarioImporter: 未対応のフォーマットです")
        return

    for timeline in data.get("timelines", []):
        _create_timeline(timeline)

    print("ScenarioImporter: インポート完了")

static func _create_timeline(timeline: Dictionary) -> void:
    var tid    = timeline.get("id", "unknown")
    var events = timeline.get("events", [])
    var lines  = ["[Dialogic Timeline]", "version = \"2.0\"", ""]

    for ev in events:
        match ev.get("event_type"):
            "dialogic_text":
                lines.append("[dialogic_event_text]")
                lines.append("text = %s" % JSON.stringify(ev.get("text", "")))
            "dialogic_character":
                lines.append("[dialogic_event_character]")
                lines.append("character = \"%s%s.dch\"" % [DIALOGIC_CHARACTER_PATH, ev.get("character", "")])
                lines.append("portrait = \"%s\"" % ev.get("portrait", "default"))
                lines.append("action = %d" % (1 if ev.get("action") == "leave" else 0))
                lines.append("position = %d" % (2 if ev.get("position") == "right" else 1))
            "dialogic_choice":
                lines.append("[dialogic_event_choice]")
                lines.append("question = %s" % JSON.stringify(ev.get("question", "")))
                var opts = ev.get("options", [])
                for i in range(opts.size()):
                    lines.append("option_%d = %s" % [i, JSON.stringify(opts[i].get("text", ""))])
                    if opts[i].get("timeline"):
                        lines.append("option_%d_timeline = \"%s%s.dtl\"" % [i, DIALOGIC_TIMELINE_PATH, opts[i].get("timeline")])
            "dialogic_jump":
                lines.append("[dialogic_event_jump]")
                lines.append("timeline = \"%s%s.dtl\"" % [DIALOGIC_TIMELINE_PATH, ev.get("timeline", "")])
        lines.append("")

    var out_path = "%s%s.dtl" % [DIALOGIC_TIMELINE_PATH, tid]
    DirAccess.make_dir_recursive_absolute(DIALOGIC_TIMELINE_PATH.get_base_dir())
    var out = FileAccess.open(out_path, FileAccess.WRITE)
    if out:
        out.store_string("\n".join(lines))
        out.close()
        print("ScenarioImporter: 書き出し完了: " + out_path)
    else:
        push_error("ScenarioImporter: 書き込み失敗: " + out_path)
```

> **注意**: Dialogic 2 のイベントフォーマットはバージョンによって異なる場合があります。  
> Dialogic 側の仕様変更があった場合は GDScript の `_create_timeline()` 内の `lines.append()` 部分を修正してください。

---

## ファイルフォーマット（ツール用 JSON）

```json
{
  "nodes": [
    {
      "id": "text_123", "type": "text",
      "rawText": "本文", "label": "本文",
      "x": 0, "y": 0, "fixed": false
    },
    {
      "id": "sprite_456", "type": "sprite",
      "charName": "Alice", "spritePos": "left",
      "expression": "smile", "spriteAction": "enter",
      "label": "Alice\n左 | 笑顔\n登場",
      "rawText": "", "x": 200, "y": 0, "fixed": false
    },
    {
      "id": "choice_789", "type": "choice",
      "rawText": "どうする？", "label": "どうする？",
      "childIds": ["opt_choice_789_1", "opt_choice_789_2"],
      "childOffsets": {
        "opt_choice_789_1": { "x": 240, "y": -55 },
        "opt_choice_789_2": { "x": 240, "y":  55 }
      },
      "childCount": 2, "x": 400, "y": 0, "fixed": false
    }
  ],
  "edges": [
    { "from": "text_123",   "to": "sprite_456" },
    { "from": "sprite_456", "to": "choice_789" }
  ]
}
```

### SPRITE ノードのプロパティ

| プロパティ | 値 |
|---|---|
| `charName` | キャラクター名（文字列）|
| `spritePos` | `"left"` または `"right"` |
| `expression` | `default` / `smile` / `confused` / `surprised` / `shy` / `angry` / `sad` / `cry` / `gag` |
| `spriteAction` | `"enter"` または `"exit"` |

---

## 読込時の安全性

- ファイルサイズ上限: **2 MB**
- ノード数上限: **500**、エッジ数上限: **2000**
- `type` は `text / choice / option / sprite` のみ許可
- 文字列から制御文字を除去
- 数値は正当な範囲にクランプ
- 存在しないノード ID を参照するエッジはエラー
- 自己ループはエラー

---

## ブラウザ要件

- Chrome / Edge 最新版
- Firefox 最新版
- Safari 16 以降
