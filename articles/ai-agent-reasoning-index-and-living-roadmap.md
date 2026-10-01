# AIエージェントのRoutingを軽量化するReasoning Indexと、計画を捨てられるLiving Roadmap

AIエージェントへSkill、Memory、Ruleを追加していくと、知識量だけでなく「何を読むか決めるための情報」も増えます。

この文書は、次の二つを再現できる形でまとめた実装ノートです。

1. **Compact Reasoning Index**  
   大きなSkill / Memory Registryを毎回全文展開せず、候補選択だけを小さな派生Indexで行う。
2. **Living Roadmap**  
   自己改善Featureを固定Phaseで完走せず、実測やModel / Host / Toolの変化で計画そのものを再導出する。

実運用では、Skill routing surfaceを **38,533文字 → 10,657文字（72.3%減）**、Memory routing surfaceを **33,301文字 → 6,678文字（79.9%減）** まで縮小しました。

この数字はtokenやlatencyの改善率ではありません。静的なrouting inputの文字数比較です。

---

## 1. 解きたい問題

Agent Harnessが次の構造を持っているとします。

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

Skill本文を必要時だけ読むようにしても、Skill Registry自体が大きくなると、Fast Routerで閉じないTaskのたびに大きなRegistryを読むことになります。

ここでやりたいことはSemantic Searchの最大化ではありません。

**正規情報源を保ったまま、最初の候補探索面だけを小さくすること**です。

---

## 2. 設計上の不変条件

この実装で最優先するのは次です。

### Canonical sourceを唯一のAuthorityにする

Generated Indexは候補発見用です。

```text
generated index != source of truth
```

Indexと正規Registryが矛盾した場合は、必ず正規Registryを優先します。

### staleなら使わない

Index生成時のsource revisionを保存します。

現在のsourceと一致しなければ、Indexを破棄してfull Registryへfallbackします。

### ambiguousならfull Registryへ戻る

Compact Indexだけで必要Skill集合を閉じられない場合、無理に推測しません。

### task-critical Skill本文は別問題

候補選択をcompact化しても、選ばれたSkill本文のSafety / completion / fallback等を落とさないよう、最初は全文取得を維持します。

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

| File | Role |
| --- | --- |
| `skills.yaml` | canonical Skill routing authority |
| `memory.yaml` | canonical Memory routing authority |
| `routing.min.json` | runtime candidate selection |
| `catalog.json` | human / debug / inspection |
| Skill / Memory Markdown | canonical body |

---

## 4. Canonical Skill Registry

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

## 5. Runtime Index schema

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

### なぜtupleにするか

次のようなpretty objectは読みやすいですが、runtime用途ではfield nameが毎entry重複します。

```json
{
  "id": "research",
  "source_start": 2,
  "source_end": 8,
  "triggers": ["research", "latest", "source"]
}
```

Runtime fileは機械向けなので、schemaを先頭に一度だけ置いてtuple化します。

---

## 6. Git blob SHAでfreshnessを見る

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

Runtime:

```go
func CanUseIndex(indexSHA, currentSHA string) bool {
    return indexSHA != "" && indexSHA == currentSHA
}
```

一致しなければfallbackします。

---

## 7. Candidate routing

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
  ↓ candidate
canonical registry range
  ↓ composition / dependency / forced rule
required closure
  ↓
canonical Skill bodies
```

Compact Indexだけで最終決定しないことが重要です。

---

## 8. Canonical line rangeを使う

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

Candidateが `research` なら2〜8行だけ取り直します。

そこで初めて、

- path
- priority
- status
- provides
- requires
- canonical metadata

等を読みます。

---

## 9. Fallback matrix

最低限、次を決めておきます。

| Condition | Action |
| --- | --- |
| Index missing | full Registry |
| invalid JSON | full Registry |
| source SHA mismatch | full Registry |
| no candidate | full Registry or normal search |
| multiple ambiguous candidates | canonical Registry |
| high-risk Task | canonical required closure |
| schema unknown | full Registry |

Pseudo code:

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

## 10. RuntimeとInspectionを分離する

最初の実装では、debugに便利な情報をRuntime JSONへ入れすぎました。

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

## 11. size regressionを入れる

Runtime Indexが少しずつ肥大化しないようにします。

```go
func ValidateCompact(runtime, canonical []byte) error {
    // 70%未満を構造的なRegression gateとして使う例
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

自分のHarnessで「このIndexを入れる価値がある」と言える上限を決めます。

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

Mainでgenerated fileを自動commitするなら、push直前にmainが進んでいないか確認してください。

古いcheckoutが新しいcanonical stateを上書きしないためです。

---

# Living Roadmap

ここからは、このIndexをどう育てるかです。

自己改善Featureを固定Roadmapで管理すると、実測で前提が外れてもPhaseを完走しがちです。

そこでFeatureごとに次を持たせます。

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

## 重要: promotion criteriaは自動昇格条件ではない

条件を満たしたらstableに自動変更する、という意味ではありません。

```text
criteria satisfied
  ↓
now there is enough evidence to reconsider promotion
```

です。

環境が変わっていれば、Featureを閉じる判断もできます。

---

## Roadmap replanの実例

今回、最初のIndexは正しく動きました。

しかし27,103文字ありました。

これは設計時に想定した削減より弱かったため、次Phaseへ進まず、現在Phaseそのものを再設計しました。

```text
initial implementation
  ↓
measure
  ↓
goal gap detected
  ↓
replan
  ↓
split runtime / inspection
  ↓
measure again
```

結果:

```text
27,103 → 10,657 chars
```

ここで重要なのは、Featureを完成させることがGoalではなかった点です。

Goalは、

> AIを使いやすく、高性能にする

ことでした。

---

# 他のAIへ渡す実装指示

別のAI Agentへこの方式を実装させる場合、次をそのままTask Contractとして渡せます。

```yaml
objective:
  - canonical Skill/Memory authorityを維持する
  - first-read routing surfaceを小さくする

deliverables:
  - deterministic generator
  - skills/routing.min.json
  - memory/routing.min.json
  - verbose inspection catalogs
  - freshness validator
  - stale/ambiguous fallback
  - CI
  - size regression
  - rollout roadmap

constraints:
  - generated index is never authority
  - source revision must be verifiable
  - no silent stale usage
  - task-critical Skill body behavior must not be weakened in v1
  - no vector DB required for v1

verification:
  - generator deterministic
  - stale source detected
  - invalid index falls back
  - candidate returns to canonical registry
  - runtime index materially smaller than source
  - existing routing regression suite still passes

rollout:
  skill_routing: limited_live
  memory_routing: shadow
  section_loading: disabled
```

## 実装AIへの注意

次を勝手に最適化しないよう明示してください。

- Indexを正本に変えない
- Canonical pathを削除しない
- stale時に推測で継続しない
- compact化のついでにSkill本文まで部分読みしない
- token削減をcorrectnessより上位に置かない
- debug catalogをruntime first-readへ戻さない

---

# Acceptance

実装完了は次で判定できます。

- [ ] canonical Registryが残っている
- [ ] generated Indexが再生成可能
- [ ] source SHAを比較できる
- [ ] stale時にfallbackする
- [ ] ambiguous時にfallbackする
- [ ] candidateからcanonical entryへ戻れる
- [ ] required Skill closureを既存Routerで確定する
- [ ] task-critical bodyのSafety / completion contractを落としていない
- [ ] runtimeとinspectionが分離されている
- [ ] size regressionがある
- [ ] CIが通る
- [ ] Featureにnext_probe / promotion / demotion / replan条件がある

---

# どこまで一般化できるか

この方式が向くのは、検索対象にすでに構造がある場合です。

- explicit ID
- trigger
- section
- canonical registry
- stable source files
- deterministic generation

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

重要なのは、検索方式よりSource of Truthを先に決めることです。

---

# 最後に

AIエージェントが大きくなると、問題は「知識が足りない」だけではなくなります。

**必要な知識へどう到達するか。**

そして自己改善を続けると、

**今の到達方法をいつ捨てるか。**

も必要になります。

Compact Reasoning Indexは前者を扱います。

Living Roadmapは後者を扱います。

この二つを組み合わせると、

```text
small discovery
  ↓
canonical truth
  ↓
execution
  ↓
measurement
  ↓
keep / promote / demote / redesign
```

という循環を作れます。

私にとって今回の改善で一番価値があったのは、Indexを作ったことではありません。

**実測が弱かったとき、自分で作ったRoadmapを捨てる仕組みが、その場で実際に働いたこと**でした。

## 関連記事

- https://zenn.dev/c_a_p_engineer/articles/ai-agent-skill-routing
- https://zenn.dev/c_a_p_engineer/articles/obsidian-reasoning-index
- https://github.com/VectifyAI/PageIndex
