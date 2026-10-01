# AIエージェントのRoutingを軽量化するReasoning Indexと、見直せる改善ロードマップ

AIエージェントへSkill、Memory、Ruleを追加していくと、知識量だけでなく「何を読むか決めるための情報」も増えます。

この文書は、次の二つを再現できる形でまとめた実装ノートです。

1. **軽量Reasoning Index（Compact Reasoning Index）**  
   大きなSkill / Memory Registryを毎回全文展開せず、候補選択だけを小さな派生Indexで行う。
2. **見直せる改善ロードマップ**  
   自己改善の機能を決めた段階どおりに消化するのではなく、実測結果やModel / 実行環境（Host） / Toolの変化に応じて、計画そのものを組み直す。

実運用では、Routing時に最初に読む情報量を、Skill側で **38,533文字 → 10,657文字（72.3%減）**、Memory側で **33,301文字 → 6,678文字（79.9%減）** まで減らしました。

この数字は、Token数やLatencyが同じ割合で改善したという意味ではありません。比較しているのは、Routing時に最初に読むテキストの文字数です。

---

## 1. 解きたい問題

AIエージェントを動かす周辺の仕組み（Agent Harness）が、次の構造を持っているとします。

```text
Bootstrap
  ↓
Fast Router
  ↓
Skill Registry
  ↓
Skill bodies
  ↓
Memory Index
  ↓
Memory bodies
```

Skill本文を必要時だけ読むようにしても、Skill Registry自体が大きくなると、Fast Routerだけでは必要なSkillを決められないタスクのたびに、大きなRegistryを読むことになります。

ここでやりたいことはSemantic Searchの最大化ではありません。

**正規の情報源を保ったまま、候補を探すために最初に読む情報量だけを小さくすること**です。

---

## 2. 設計上の不変条件

この実装で最優先するのは次です。

### 正規の情報源を一つに決める

生成したIndexは、候補を見つけるためだけに使います。

```text
生成Index ≠ 正本（Source of Truth）
```

Indexと正規Registryが食い違った場合は、必ず正規Registryを正しいものとして扱います。

### 元データより古いIndexは使わない

Index生成時のsource revisionを保存します。

現在の元データと一致しなければ、そのIndexは使わず、正規Registryの全文を読む経路へ戻ります。

### 候補を絞れない場合はRegistry全文へ戻る

軽量Indexだけで必要なSkill集合を確定できない場合は、無理に推測しません。

### そのタスクに必須のSkill本文は別に扱う

候補選択を軽量化しても、選ばれたSkill本文にある安全上の制約、完了条件、問題が起きたときの戻り先などを落とさないよう、最初は本文を全文読みます。

---

## 3. 推奨ディレクトリ構造

```text
agent/
├─ bootstrap.md
├─ registry/
│  ├─ skills.yaml
│  └─ memory.yaml
├─ skills/
│  ├─ research.md
│  ├─ coding.md
│  └─ verification.md
├─ memory/
│  └─ deploy-lessons.md
└─ .ai-index/
   ├─ manifest.json
   ├─ skills/
   │  ├─ routing.min.json
   │  └─ catalog.json
   └─ memory/
      ├─ routing.min.json
      └─ catalog.json
```

役割は明確に分けます。

| ファイル | 役割 |
| --- | --- |
| `skills.yaml` | Skill Routingの正本 |
| `memory.yaml` | Memory Routingの正本 |
| `routing.min.json` | 実行時の候補選択 |
| `catalog.json` | 人間による確認・デバッグ |
| Skill / Memory Markdown | 正規の本文 |

---

## 4. 正規のSkill Registry

例:

```yaml
skills:
  research:
    path: skills/research.md
    priority: P0
    triggers:
      - research
      - latest
      - source

  coding:
    path: skills/coding.md
    priority: P0
    triggers:
      - implementation
      - bugfix
      - code

  verification:
    path: skills/verification.md
    priority: P0
    triggers:
      - test
      - verify

composition:
  software_change:
    - coding
    - verification
```

Memory:

```yaml
entries:
  deploy-lessons:
    path: memory/deploy-lessons.md
    type: procedural
    load_policy: on_match
    triggers:
      - deploy
      - release
```

ここにすでにIDとTriggerがあるなら、その構造を先に使います。

---

## 5. 実行時用Indexのスキーマ

Skill側:

```json
{
  "v": 1,
  "authority": "registry/skills.yaml",
  "source_git_blob_sha": "012345...",
  "skill_fields": [
    "id",
    "source_start",
    "source_end",
    "triggers"
  ],
  "skills": [
    ["research", 2, 8, ["research", "latest", "source"]],
    ["coding", 9, 15, ["implementation", "bugfix", "code"]]
  ],
  "composition_fields": [
    "id",
    "source_start",
    "source_end"
  ],
  "compositions": [
    ["software_change", 22, 25]
  ]
}
```

Memory側:

```json
{
  "v": 1,
  "authority": "registry/memory.yaml",
  "source_git_blob_sha": "abcdef...",
  "runtime_status": "shadow",
  "fields": [
    "id",
    "source_start",
    "source_end",
    "triggers"
  ],
  "entries": [
    ["deploy-lessons", 2, 9, ["deploy", "release"]]
  ]
}
```

### なぜ配列形式（tuple）にするか

次のように整形したobjectは人間には読みやすい一方、実行時用途ではフィールド名が項目ごとに繰り返されます。

```json
{
  "id": "research",
  "source_start": 2,
  "source_end": 8,
  "triggers": ["research", "latest", "source"]
}
```

実行時用ファイルは機械が読むためのものなので、スキーマを先頭に一度だけ置き、各項目はtupleで持ちます。

---

## 6. Git blob SHAでIndexの鮮度を確認する

Git blob SHA-1:

```go
package index

import (
    "crypto/sha1"
    "encoding/hex"
    "fmt"
)

func GitBlobSHA(data []byte) string {
    h := sha1.New()
    _, _ = fmt.Fprintf(h, "blob %d%c", len(data), byte(0))
    _, _ = h.Write(data)
    return hex.EncodeToString(h.Sum(nil))
}
```

Index生成時:

```go
sourceSHA := GitBlobSHA(registryBytes)
```

実行時:

```go
func CanUseIndex(indexSHA, currentSHA string) bool {
    return indexSHA != "" && indexSHA == currentSHA
}
```

一致しなければ正規Registryへ戻ります。

---

## 7. 候補となるSkillを選ぶ

最小Router:

```go
type SkillTuple struct {
    ID       string
    Start    int
    End      int
    Triggers []string
}

func Match(task string, skills []SkillTuple) []SkillTuple {
    task = strings.ToLower(task)

    out := make([]SkillTuple, 0)

    for _, skill := range skills {
        for _, trigger := range skill.Triggers {
            if strings.Contains(task, strings.ToLower(trigger)) {
                out = append(out, skill)
                break
            }
        }
    }

    return out
}
```

これは高度なSemantic Routerではありません。

重要なのは、候補を決めた後です。

```text
routing.min
  ↓ 候補を絞る
正規Registryの該当範囲
  ↓ Composition / 依存関係 / 強制適用Ruleを確認
必要なSkill集合を確定
  ↓
正規のSkill本文
```

軽量Indexだけで最終決定しないことが重要です。

---

## 8. 正規Registryの行範囲を使う

Indexにpathまで複製せず、Registry上のentry範囲だけ保存できます。

```text
skills:
  research:      ← 2
    ...
                  ← 8
  coding:        ← 9
    ...
                  ← 15
```

候補が `research` なら、正規Registryの2〜8行だけを読み直します。

そこで初めて、

- path
- priority
- status
- provides
- requires
- 正規のメタデータ

等を読みます。

---

## 9. 正規Registryへ戻る条件

最低限、次を決めておきます。

| Condition | Action |
| --- | --- |
| Indexがない | 正規Registry全文を読む |
| JSONが壊れている | 正規Registry全文を読む |
| 元データのSHAが一致しない | 正規Registry全文を読む |
| 候補が見つからない | 正規Registry全文、または通常検索へ戻る |
| 候補を一つに絞れない | 正規Registryで確認する |
| 高リスクなタスク | 正規Registryで必要なSkill集合を確定する |
| 未対応のschema | 正規Registry全文を読む |

疑似コード:

```go
func Route(task string) Result {
    idx, err := loadIndex()
    if err != nil {
        return routeFullRegistry(task)
    }

    currentSHA := currentRegistrySHA()
    if idx.SourceSHA != currentSHA {
        return routeFullRegistry(task)
    }

    candidates := idx.Match(task)
    if !CanCloseRequiredSet(candidates) {
        return routeFullRegistry(task)
    }

    entries := readCanonicalRanges(candidates)
    closure := ResolveClosure(entries)

    return LoadCanonicalBodies(closure)
}
```

---

## 10. 実行時用と確認・デバッグ用を分離する

最初の実装では、デバッグに便利な情報まで実行時用JSONへ入れすぎました。

結果:

```text
canonical registry  38,533 chars
verbose catalog     27,103 chars
```

約30%減にしかなりません。

そこで分けました。

```text
routing.min.json
  - first read
  - IDs
  - triggers
  - source ranges
  - source SHA

catalog.json
  - path
  - priority
  - status
  - hashes
  - complete composition
  - debug metadata
```

結果:

```text
38,533 → 10,657 chars
```

72.3%減です。

Memory側は:

```text
33,301 → 6,678 chars
```

79.9%減でした。

---

## 11. Indexの肥大化を回帰テストで検出する

実行時用Indexが少しずつ肥大化しないようにします。

```go
func ValidateCompact(runtime, canonical []byte) error {
    // 70%未満をサイズの回帰テスト条件として使う例
    if len(runtime)*10 >= len(canonical)*7 {
        return fmt.Errorf(
            "runtime index too large: runtime=%d canonical=%d",
            len(runtime),
            len(canonical),
        )
    }
    return nil
}
```

70%に一般的な意味はありません。

自分のAgent Harnessで「このIndexを入れる価値がある」と言える上限を決めます。

---

## 12. CI

```yaml
name: validate-agent-index

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: "1.24"

      - name: Test
        run: go test ./...

      - name: Build indexer
        run: go build -o /tmp/indexer ./cmd/indexer

      - name: Generate
        run: /tmp/indexer generate --write

      - name: Validate
        run: /tmp/indexer validate

      - name: Freshness
        run: /tmp/indexer generate --check

      - name: Generated diff
        run: git diff --exit-code -- .ai-index/
```

mainで生成ファイルを自動commitするなら、push直前にmainブランチが先へ進んでいないか確認してください。

古いcheckoutから作った生成物が、新しい正規データを上書きしないためです。

---

# 見直せる改善ロードマップ

ここからは、このIndexをどう育てるかです。

自己改善の機能を固定ロードマップで管理すると、実測で前提が外れても、決めた段階を最後まで消化しがちです。

そこで、判断に迷う改善項目には次の情報を持たせます。

YAMLのキー名は英語ですが、意味は次のとおりです。

- `next_probe`: 次に何を確認するか
- `promotion_criteria`: 次へ進めるか判断する条件
- `demotion_criteria`: 一段戻す、または止めるか判断する条件
- `replan_triggers`: 計画そのものを作り直すきっかけ

```yaml
core_reasoning_index:
  status: limited_live

  objective: >
    correctnessを維持したまま
    routing context frictionを減らす

  next_probe: >
    representative tasksで
    full-registry routingとpaired comparisonする

  promotion_criteria:
    - required skill recall >= baseline
    - stale fallback works
    - natural tasks show material efficiency gain

  demotion_criteria:
    - activation failures increase
    - stale index overrides canonical state
    - tool or maintenance cost exceeds benefit

  replan_triggers:
    - host-native retrieval materially improves
    - model context behavior changes
    - registry schema changes
    - actual index size differs materially from projection
    - top-level goal changes

  next_state_candidates:
    strong_evidence: stable
    incomplete: limited_live
    regression: observe
    no_value: closed
    architecture_changed: redesign
```

## 重要: 「進める条件」は自動昇格のスイッチではない

条件を満たしたら自動的に `stable` へ変更する、という意味ではありません。

```text
条件を満たす
  ↓
次へ進めるか再判断できるだけの根拠が揃う
```

です。

環境が変わっていれば、その機能を終了する判断もできます。

---

## ロードマップを見直した実例

今回、最初のIndexは正しく動きました。

しかし27,103文字ありました。

これは設計時に想定した削減より弱かったため、次の段階へ進まず、現在の段階そのものを再設計しました。

```text
初期実装
  ↓
実測
  ↓
目的との差を確認
  ↓
計画を見直す
  ↓
実行時用と確認用を分離
  ↓
再度実測
```

結果:

```text
27,103 → 10,657 chars
```

ここで重要なのは、その機能を完成させること自体が目的ではなかった点です。

目的は、

> AIを使いやすく、高性能にする

ことでした。

---

# 他のAIへ渡す実装指示

別のAIエージェントへこの方式を実装させる場合、次をそのまま実装契約（Task Contract）として渡せます。

```yaml
objective:
  - 正規のSkill / Memoryを唯一の正しい情報源として維持する
  - Routing時に最初に読む情報量を小さくする

deliverables:
  - 同じ入力から同じ結果を出すgenerator
  - skills/routing.min.json
  - memory/routing.min.json
  - 確認・デバッグ用の詳細Catalog
  - 生成元との一致を確認するvalidator
  - Indexが古い、または候補を絞れない場合の戻り先
  - CI
  - サイズの回帰テスト
  - 段階的な導入計画

constraints:
  - 生成Indexを正本にしない
  - 生成元のrevisionを検証できるようにする
  - 古いIndexを黙って使わない
  - v1では、そのタスクに必須のSkill本文の読み方を弱めない
  - v1ではVector DBを必須にしない

verification:
  - generatorが同じ入力から同じ結果を出す
  - 元データとの差分を検出できる
  - 壊れたIndexなら正規Registryへ戻る
  - 候補を見つけた後に正規Registryへ戻る
  - 実行時用Indexが元データより十分小さい
  - 既存のRouting回帰テストが通る

rollout:
  skill_routing: limited_live
  memory_routing: shadow
  section_loading: disabled
```

## 実装AIへの注意

次を勝手に最適化しないよう明示してください。

- Indexを正本に変えない
- 正規データへのpathを削除しない
- Indexが古い場合に推測で処理を続けない
- Indexの軽量化と同時に、Skill本文まで勝手に部分読みへ変えない
- Token削減を正しさより優先しない
- デバッグ用Catalogを、実行時に最初から読ませない

---

# 完了条件

実装完了は次で判定できます。

- [ ] 正規Registryが残っている
- [ ] 生成したIndexを正規データから再生成できる
- [ ] 生成元のSHAを比較できる
- [ ] Indexが古い場合は正規Registryへ戻る
- [ ] 候補を一つに絞れない場合は正規Registryへ戻る
- [ ] 候補から正規Registryのentryへ戻れる
- [ ] 必要なSkill集合を既存Routerで確定できる
- [ ] そのタスクに必須のSkill本文から、安全条件や完了条件を落としていない
- [ ] 実行時用と確認・デバッグ用が分離されている
- [ ] サイズの回帰テストがある
- [ ] CIが通る
- [ ] 判断が難しい改善項目に、次の観測・進める条件・戻す条件・計画を見直す条件がある

---

# どこまで一般化できるか

この方式が向くのは、検索対象にすでに構造がある場合です。

- 明示的なID
- 検索用trigger
- section構造
- 正規Registry
- 安定して参照できる元ファイル
- 同じ入力から同じ結果を生成できること

逆に、自然文だけが大量にあり、概念上の近さで横断検索したいならEmbedding / Vector Searchが強くなります。

両方を組み合わせても構いません。

```text
Fast Router
  ↓
Structural Compact Index
  ↓
Semantic Search if needed
  ↓
Canonical Source
```

重要なのは、検索方式を決める前に、どのデータを正本（Source of Truth）とするか決めておくことです。

---

# 最後に

AIエージェントが大きくなると、問題は「知識が足りない」だけではなくなります。

**必要な知識へどう到達するか。**

そして自己改善を続けると、

**今の到達方法をいつ捨てるか。**

も必要になります。

軽量Reasoning Indexは前者を扱います。

見直せる改善ロードマップは後者を扱います。

この二つを組み合わせると、

```text
小さな索引で候補を絞る
  ↓
正本で確認する
  ↓
実行する
  ↓
結果を測る
  ↓
維持 / 進める / 戻す / 再設計
```

という循環を作れます。

私にとって今回の改善で一番価値があったのは、Indexを作ったことではありません。

**実測が想定を下回ったとき、自分で作ったロードマップを捨てて設計を組み直す仕組みが、その場で実際に働いたこと**でした。

## 関連記事

- https://zenn.dev/c_a_p_engineer/articles/ai-agent-skill-routing
- https://zenn.dev/c_a_p_engineer/articles/obsidian-reasoning-index
- https://github.com/VectifyAI/PageIndex
