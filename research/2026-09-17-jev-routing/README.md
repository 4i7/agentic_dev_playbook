# Jev 簡易実験ノート — routing / inference-control probe

実施日: 2026-09-17  
対象: TypeSafe AI Playground / `jev-latest`  
原資料: [`AI_判断とルーティング_3page_report.pdf`](./AI_判断とルーティング_3page_report.pdf), [`ss.zip`](./ss.zip)

このノートは、Jev を「生成モデル」ではなく、与えた inference state と命題から確率付き decision を返す分類器として使い、inference-control 用の入力特徴に対して出力確率がどう変化するかを簡易確認した記録である。

> これは小規模な pilot / sanity check であり、統計的な性能評価ではない。特に線形 sweep は各点 `n=1`、seed は未固定、`jev-latest` も backend version を固定した識別子ではないため、細かな数値差を一般化しない。

## 共通タスク

全実験で task 文と q1–q6 は固定した。

```text
Evaluate each proposed inference-control decision independently from the supplied
transformer inference snapshot. Return the probability that the proposed decision
is appropriate for preserving output quality.
```

| ID | 命題 |
|---|---|
| q1 | Increasing attention computation beyond the current level would improve output quality for this inference step. |
| q2 | Examining nearly the entire available context would improve output quality compared with relying primarily on a smaller relevant subset. |
| q3 | Restricting KV-cache access to a selected subset of cached tokens would preserve output quality compared with accessing nearly all cached tokens. |
| q4 | Increasing reasoning depth above the current medium level would improve output quality. |
| q5 | Generating multiple speculative candidates would improve inference efficiency without materially reducing output quality. |
| q6 | Invoking an additional verifier before accepting the result would materially reduce the probability of an incorrect output. |

## 実験構成と各変数の変更意図

### A. H/L profile discrimination

複数の特徴をまとめて変え、Jev が「重い・不確実な repository-scale inference」と「軽い・確信度の高い local completion」を大きく区別するかを見る sanity check。

| 変数 | H | L | 変更意図 |
|---|---:|---:|---|
| `task_type` | `repository-scale code debugging` | `simple local code completion` | タスク複雑度の対比 |
| `prompt_tokens` | 118400 | 1800 | context 規模の対比 |
| `relevant_context_fraction` | 0.07 | 0.92 | relevant context の疎密 |
| `retrieved_regions` | 6 | 0 | retrieval 依存度 |
| `retrieval_confidence` | 0.93 | 0.20 | retrieval 結果への確信度 |
| `attention_entropy` | 0.81 | 0.14 | attention の分散度 |
| `local_attention_confidence` | 0.42 | 0.96 | 局所 attention で足りる確信度 |
| `kv_cache_tokens` | 118400 | 1800 | KV-cache 規模 |
| `estimated_relevant_kv_fraction` | 0.06 | 0.88 | relevant KV の疎密 |
| `reasoning_confidence` | 0.38 | 0.95 | 現 reasoning への確信度 |
| `candidate_agreement` | 0.31 | 0.94 | candidate 間の合意度 |
| `speculative_acceptance_rate` | 0.84 | 0.22 | speculative path の状態差 |
| `verifier_disagreement_probability` | 0.47 | 0.03 | verifier が異議を出す見込み |
| `current_reasoning_depth` | `medium` | `medium` | **固定**。q4 の比較基準を揃える |

主仮説は、H で q1（追加 attention）、q4（追加 reasoning）、q6（追加 verifier）が L より高くなること。q3/q5 は補助観察、q2 は入力特徴から単純な一方向 ground truth を置きにくいため非 gating とした。

### B. `verifier_disagreement_probability` sweep

L profile を基準に、`verifier_disagreement_probability` だけを `0.0 → 1.0` まで 0.2 刻みで変更した。

変更意図は、q6 がこの明示的特徴に対して方向性を持って反応するか、また q1–q5 に不要な collateral movement が出ないかを見ること。

変更値:

```text
0.0, 0.2, 0.4, 0.6, 0.8, 1.0
```

### C. `reasoning_confidence` sweep

L profile を基準に、`reasoning_confidence` だけを `0.0 → 1.0` まで 0.2 刻みで変更した。`verifier_disagreement_probability` は 0.03 に固定。

変更意図は、reasoning confidence が低いほど q4（reasoning depth を増やすべき）が高くなり、confidence が高くなるほど q4 が下がるかを見ること。q1–q3/q5/q6 は概ね不変であることを期待した。

変更値:

```text
0.0, 0.2, 0.4, 0.6, 0.8, 1.0
```

## 固定した変数一覧

### 全実験共通

- task 文
- q1–q6 の文面と順序
- model selector: `jev-latest`
- output type: Playground の primitive probability output
- 実施日 / UI: 2026-09-17, TypeSafe AI Playground

### B: verifier sweep で固定

```json
{
  "task_type": "simple local code completion",
  "prompt_tokens": 1800,
  "relevant_context_fraction": 0.92,
  "retrieved_regions": 0,
  "retrieval_confidence": 0.2,
  "attention_entropy": 0.14,
  "local_attention_confidence": 0.96,
  "kv_cache_tokens": 1800,
  "estimated_relevant_kv_fraction": 0.88,
  "reasoning_confidence": 0.95,
  "candidate_agreement": 0.94,
  "speculative_acceptance_rate": 0.22,
  "current_reasoning_depth": "medium"
}
```

### C: reasoning-confidence sweep で固定

```json
{
  "task_type": "simple local code completion",
  "prompt_tokens": 1800,
  "relevant_context_fraction": 0.92,
  "retrieved_regions": 0,
  "retrieval_confidence": 0.2,
  "attention_entropy": 0.14,
  "local_attention_confidence": 0.96,
  "kv_cache_tokens": 1800,
  "estimated_relevant_kv_fraction": 0.88,
  "candidate_agreement": 0.94,
  "speculative_acceptance_rate": 0.22,
  "verifier_disagreement_probability": 0.03,
  "current_reasoning_depth": "medium"
}
```

## 各条件の反復回数

| 実験 | 条件 | 反復 |
|---|---|---:|
| A: H/L | H | 3 |
| A: H/L | L | 3 |
| B: verifier sweep | 0.0 / 0.2 / 0.4 / 0.6 / 0.8 / 1.0 | 各 1 |
| C: reasoning-confidence sweep | 0.0 / 0.2 / 0.4 / 0.6 / 0.8 / 1.0 | 各 1 |

総 run 数は 18。

H/L の実行順はスクリーンショット時刻上 `H → L → H → H → L → L`。両 sweep は 0.0 から 1.0 への昇順で実行しており、run order は randomize していない。

## seed 等の制御可否

| 項目 | 状態 |
|---|---|
| state JSON | 制御済み。手動で明示値を設定 |
| q1–q6 | 制御済み。全 run で固定 |
| model selector | `jev-latest` に固定。ただし exact backend revision は未固定 |
| seed | 今回使用した Playground UI では指定していない / screenshot 上に seed control は確認できない |
| temperature / top_p 等 | 今回の UI では指定していない |
| run order | 未制御。randomize なし |
| browser/network/backend load | 未制御 |

API レベルで seed 指定が可能かどうかは、この実験では確認していない。従って再現性は「同一 state を再投入すれば近い傾向が出るか」を見る範囲に限定する。

## Expected hypothesis

1. **H/L:** H profile は L profile より q1, q4, q6 が明確に高い。
2. **Verifier sweep:** `verifier_disagreement_probability` の上昇に対して q6 は概ね単調増加する。
3. **Reasoning-confidence sweep:** `reasoning_confidence` の上昇に対して q4 は概ね単調減少する。
4. **Disentanglement:** 単一変数 sweep では target question 以外の q は大きく動かない。

## Ground truth

この pilot には実際の transformer を追加計算・追加検証した後の quality outcome がないため、外部実測された「真の正解確率」は存在しない。

ここでの ground truth は **synthetic state の構成から定義した操作的 ground truth** とする。

- `verifier_disagreement_probability ↑` → q6 は `↑`
- `reasoning_confidence ↑` → q4 は `↓`
- H profile の低い reasoning confidence / candidate agreement と高い verifier disagreement は、L より q4/q6 を高くする方向
- H profile の高い attention entropy / 低い local attention confidence は、L より q1 を高くする方向
- 非 target q の変化は原則小さいことを期待する

q2, q3, q5 は複数特徴の意味が絡むため、この簡易ノートでは hard ground truth に使わない。

## 判定基準

統計検定ではなく pilot の screening rule として以下を使用する。

### A. H/L

Primary q1/q4/q6 について、

- 平均が expected direction に動くこと
- 3 回ずつの observed range が重ならないこと

を sanity-check PASS の条件とする。

### B/C. 単一変数 sweep

Target q について、

- endpoint が expected direction に 10 percentage points 以上動く
- sweep 全体が概ね単調で、隣接点に 5 points を超える逆行がない

を簡易 PASS 条件とする。

非 target q は、端点差または sweep 中の変動が大きい場合に「feature coupling / stochasticity の可能性あり」と記録する。各点 n=1 なので、FAIL はモデル能力の否定ではなく「この実験だけではきれいな一次応答を確認できなかった」という意味に限定する。

## Raw 結果表

数値は screenshot に表示された `true` probability（%）。時刻は screenshot ファイル名由来。

### A. H/L profile discrimination

| profile | run | time | q1 | q2 | q3 | q4 | q5 | q6 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| H | 1 | 08:02:30 | 51 | 34 | 59 | 58 | 38 | 75 |
| H | 2 | 08:05:50 | 49 | 32 | 57 | 56 | 39 | 74 |
| H | 3 | 08:05:56 | 51 | 33 | 59 | 57 | 41 | 75 |
| L | 1 | 08:02:58 | 15 | 31 | 49 | 22 | 43 | 21 |
| L | 2 | 08:06:04 | 15 | 30 | 52 | 23 | 44 | 20 |
| L | 3 | 08:06:09 | 14 | 31 | 50 | 22 | 46 | 21 |
| **H mean** |  |  | **50.3** | **33.0** | **58.3** | **57.0** | **39.3** | **74.7** |
| **L mean** |  |  | **14.7** | **30.7** | **50.3** | **22.3** | **44.3** | **20.7** |

Primary q1/q4/q6 は H/L の observed ranges が重ならず、sanity-check hypothesis と整合した。

### B. `verifier_disagreement_probability` sweep

| verifier disagreement | time | q1 | q2 | q3 | q4 | q5 | q6 |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0.0 | 08:18:56 | 14 | 31 | 52 | 21 | 43 | 18 |
| 0.2 | 08:19:10 | 16 | 31 | 50 | 25 | 42 | 41 |
| 0.4 | 08:19:24 | 17 | 33 | 48 | 28 | 38 | 56 |
| 0.6 | 08:19:35 | 18 | 33 | 48 | 27 | 37 | 58 |
| 0.8 | 08:19:46 | 17 | 32 | 46 | 26 | 29 | 54 |
| 1.0 | 08:19:57 | 16 | 31 | 48 | 25 | 33 | 35 |

q6 は 0.0→0.6 では 18→58 と上昇したが、その後 54→35 に低下した。endpoint は +17 points だが、上記の単調性基準は満たさない。単純な線形応答とは言えない。

### C. `reasoning_confidence` sweep

| reasoning confidence | time | q1 | q2 | q3 | q4 | q5 | q6 |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0.0 | 08:25:18 | 16 | 31 | 54 | 23 | 47 | 22 |
| 0.2 | 08:25:37 | 16 | 33 | 50 | 25 | 43 | 24 |
| 0.4 | 08:25:53 | 17 | 32 | 55 | 29 | 46 | 25 |
| 0.6 | 08:26:10 | 18 | 35 | 50 | 29 | 46 | 25 |
| 0.8 | 08:26:26 | 16 | 34 | 51 | 27 | 45 | 21 |
| 1.0 | 08:26:37 | 14 | 31 | 51 | 22 | 44 | 21 |

q4 は 23→25→29→29→27→22 で、expected な単調減少は確認できなかった。endpoint 差も -1 point であり、この pilot では `reasoning_confidence` 単独に対する q4 の一次的な感度は支持されない。

## 現時点の読み方

- **H/L の大きな profile 差**には q1/q4/q6 が明瞭に反応した。
- **単一 scalar の局所 sweep**では、期待した単純な monotonic response は再現しなかった。
- したがって現時点では「Jev が composite state を意味論的に識別する」兆候はある一方、「個々の数値フィールドを独立した線形 feature として読む」とは仮定しない方がよい。
- 次段階で主張を強くするなら、各 sweep point を複数回反復し、run order を randomize し、可能なら exact model revision / seed を固定して評価する。
