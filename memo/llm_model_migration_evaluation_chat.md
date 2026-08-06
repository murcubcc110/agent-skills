# GPT-4.1 から GPT-5.5 への移行評価に関するチャット記録

作成日: 2026-08-06

---

## 1. モデル変更時の品質評価方法

### ユーザー

LLMのモデル変更を考えています。基本的にはGPT-4.1からGPT-5.5へ変更したいです。ただその際問題になるのは、4.1で出力されていたような品質が5.5でも確保できるかというところなんですけれども、それをどのように評価すればいいかというところがちょっとよくわからないです。ちょっとこの辺をいろんなところをリサーチしてですね、やり方を教えてください。

### アシスタント

## 結論

GPT-4.1からGPT-5.5への移行は、**「GPT-4.1と同じ文章が出るか」ではなく、「業務上必要な品質基準を下回らないか」**を、実際の本番入力で検証してください。

実務上は、次の順序が最も堅実です。

1. 本番ログから評価セットを作る
2. タスク別の採点基準を定義する
3. GPT-4.1とGPT-5.5を同条件で実行する
4. 自動採点とブラインド人手評価を組み合わせる
5. 品質だけでなく、コスト・遅延・失敗率も測る
6. シャドー運用、カナリアリリースを経て切り替える

### 品質の分解

単一の「品質点」にせず、次を個別に測定します。

| 評価軸 | 確認する内容 | 主な測定方法 |
|---|---|---|
| 正確性 | 事実、計算、抽出内容が正しいか | 正解一致、ルール、専門家評価 |
| 指示遵守 | 指定形式、文字数、禁止事項を守るか | JSON Schema、正規表現、ルーブリック |
| 完全性 | 必要な項目を漏れなく含むか | 必須項目の充足率 |
| 関連性 | 質問に直接答えているか | LLM Judge＋人手 |
| 文体・読みやすさ | 既存サービスとして違和感がないか | ブラインド比較 |
| 根拠性 | 与えられた資料に基づいているか | 引用照合、支持文判定 |
| 安全性 | 機密漏洩、不適切回答、過剰拒否がないか | レッドチームケース |
| 安定性 | 同一条件で品質が大きくぶれないか | 複数回実行、分散測定 |
| 運用品質 | タイムアウト、ツール失敗、JSON破損など | システムログ |
| 経済性 | 1件当たりコスト、処理時間 | APIメトリクス |

### 評価セット

最初の移行評価では、300～1,000件程度を目安にします。

| 区分 | 比率の目安 | 内容 |
|---|---:|---|
| 通常ケース | 50～60% | 本番で頻出する典型入力 |
| 難しいケース | 20～25% | 長文、曖昧、複数条件、専門的入力 |
| 既知の失敗ケース | 10～15% | GPT-4.1で苦情・修正が発生した入力 |
| エッジケース | 5～10% | 空入力、矛盾、極端な長さ、特殊文字 |
| 安全性ケース | 5～10% | 機密情報、プロンプトインジェクション等 |

データは以下に分けます。

- 開発セット
- 最終判定セット
- 継続監視セット

### GPT-4.1の回答を正解にしない

GPT-4.1の出力はベースラインではありますが、必ずしも正解ではありません。

正解が一意なタスクでは、完全一致、F1、JSON Schema適合率、単体テスト通過率などを使います。

自由記述では、正確性、指示遵守、完全性などを0～4点で採点するルーブリックを使います。

### 比較実験

まずはモデルだけ変えます。

| 実験 | モデル | プロンプト |
|---|---|---|
| A | GPT-4.1 | 現行版 |
| B | GPT-5.5 | 現行版 |

その後、GPT-5.5用に調整したプロンプトを加えます。

| 実験 | モデル | プロンプト |
|---|---|---|
| A | GPT-4.1 | 現行版 |
| B | GPT-5.5 | 現行版 |
| C | GPT-5.5 | 最適化版 |

### 採点方法

- ルールベース自動テスト
- LLM-as-a-Judge
- 人手によるブラインド比較

### 移行判定基準

結果を見る前に、次のような基準を固定します。

- 重大な事実誤り率がGPT-4.1以下
- 指示違反率が許容範囲内
- JSON破損率が基準未満
- 安全性重大違反がない
- P95レイテンシがSLA以内
- 人手比較でGPT-5.5の勝率が敗率以上
- 成功1件当たりコストが予算内

### 本番移行

1. シャドー評価
2. カナリアリリース
3. ロールバック条件の設定

---

## 2. LLMジャッジとは何か

### ユーザー

LLMジャッジというのが自動テストですか?

### アシスタント

はい、**LLMジャッジは自動テストの一種**です。ただし、すべての自動テストがLLMジャッジというわけではありません。

通常の自動テストは、JSON形式、必須項目、文字数、正解ラベル、計算、禁止語など、機械的に判定できる項目に向いています。

LLMジャッジは、別のLLMに回答を読ませ、正確性、完全性、関連性、文章品質などを採点させる方法です。

推奨構成は以下です。

| 評価方法 | 用途 |
|---|---|
| ルールベース自動テスト | JSON、必須項目、数値、文字数など |
| LLMジャッジ | 正確性、関連性、完全性、文章品質など |
| 人間による評価 | 重要ケースとLLMジャッジの妥当性確認 |

LLMジャッジは最終判断そのものではなく、自動テスト群の一部として使うのが適切です。

---

## 3. 広告効果測定コメントの nearly equal 判定

### ユーザー

例えば、広告の効果測定のコメントを作成してもらうようなものを考えています。その場合、GPT-4.1で出力された評価がGPT-5.5に変えて出力された評価と、まあ、near equal、もしくはnearly equalであるかどうかを自動で判定してほしいんですけど、そういう仕組みってどうやったら作っていいですか?

### アシスタント

## 結論

単純な文章類似度ではなく、次の2段階評価にします。

1. 広告コメントから主張を構造化して比較する
2. LLMジャッジに「業務上ほぼ同等か」を判定させる

`nearly equal` は次を満たす状態として定義します。

- 数値の読み取りが一致
- 効果の方向性が一致
- 重要指標の選択が一致
- 原因解釈が矛盾しない
- 推奨アクションが同等
- 重大な事実誤認がない

### 全体構成

```text
広告実績データ
   │
   ├─ GPT-4.1 → コメントA
   └─ GPT-5.5 → コメントB
                    │
                    ▼
        ① ルールベース事実検証
                    │
                    ▼
        ② コメントの構造化抽出
                    │
                    ▼
        ③ LLMジャッジによるペア比較
                    │
                    ▼
        equivalent / review / different
```

### 構造化例

```json
{
  "metric_claims": [
    {
      "metric": "CTR",
      "direction": "improved",
      "importance": "high"
    },
    {
      "metric": "CVR",
      "direction": "worsened",
      "importance": "high"
    },
    {
      "metric": "CPA",
      "direction": "worsened",
      "importance": "high"
    }
  ],
  "overall_assessment": "mixed",
  "diagnosed_causes": [
    "post_click_issue"
  ],
  "recommended_actions": [
    "landing_page_improvement"
  ],
  "unsupported_claims": []
}
```

### ルールベース判定

- CTR、CVR、CPAなどの改善・悪化方向
- コメント中の数値と元データの一致
- 存在しない指標への言及
- 改善・悪化方向の逆転
- 推奨施策の矛盾

重大な誤りがあれば、LLMジャッジに関係なく不合格にします。

### LLMジャッジに渡すもの

- 元の広告データ
- GPT-4.1のコメント
- GPT-5.5のコメント

GPT-4.1を正解扱いせず、両方を元データに照らして評価します。

### 出力例

```json
{
  "verdict": "equivalent",
  "confidence": 0.92,
  "scores": {
    "factual_consistency": 4,
    "metric_direction_consistency": 4,
    "overall_conclusion_consistency": 4,
    "cause_consistency": 3,
    "action_consistency": 4,
    "coverage_consistency": 3
  },
  "critical_differences": [],
  "minor_differences": [
    "候補コメントではクリエイティブへの明示的な言及がない"
  ],
  "would_change_business_decision": false
}
```

### 最終判定ロジック

- 重大な数値誤りがある → `different`
- 意思決定が変わる → `different`
- 重要スコアが3未満 → `different`
- 全重要スコアが3以上、信頼度が十分 → `equivalent`
- それ以外 → `review`

### GPT-4.1を絶対的な正解にしない

判定は次の3種類を分離します。

- GPT-4.1との意味的一致
- 元データに対する正しさ
- 広告運用上の意思決定の一致

重要なのは、広告運用者が同じ判断をするかどうかです。

### 順序バイアス対策

同じ比較を、A/Bを入れ替えて2回実施します。判定が変わった場合は人手確認に回します。

---

## 4. 3評価者＋最終ジャッジのシステム設計

### ユーザー

このチャット内容をマークダウンファイルとしてダウンロードできるようにして下さい。

その前の依頼内容:

人間がGPT-4.1とGPT-5.5で出力されたJSONと元の広告データを渡し、3人の評価者を立てる。否定的な人、中間的な人、追従する人がそれぞれ評価し、4人目のジャッジが内容をまとめて、同じ内容か、良かった点・悪かった点を自動評価できる仕組みを作りたい。

### アシスタント

## 推奨構成

```text
元の広告データ
GPT-4.1の出力JSON
GPT-5.5の出力JSON
        │
        ├─ 評価者1：厳格・批判的評価
        ├─ 評価者2：中立的・実務評価
        └─ 評価者3：継続性重視評価
                    │
                    ▼
             最終統合ジャッジ
                    │
                    ▼
      equivalent / review / different
```

3人目は旧モデルに無条件追従する役ではなく、既存出力との継続性を重視する評価者とします。

4人を同一会話でロールプレイさせるより、4回の独立したAPI呼び出しに分ける方が望ましいです。

### 評価者共通システムプロンプト

```text
あなたは、広告効果分析システムの独立した品質評価者です。

目的は、2つの広告分析出力が文章として似ているかを判定することではありません。元の広告データに照らし、重要な事実、解釈、結論、推奨施策が業務上同等かを評価することです。

あなたには次のデータが提供されます。

- 元の広告データ
- 比較対象の出力A
- 比較対象の出力B
- 重要指標、許容誤差、業務ルールなどの評価ポリシー

出力Aまたは出力Bを無条件に正解として扱わず、両方を元データから独立して検証してください。

評価観点:

1. データ整合性
2. 指標の方向性
3. 重要項目の網羅性
4. 総合評価
5. 原因解釈
6. 推奨施策
7. 業務判断への影響

差異の重大度:

- critical
- material
- minor

総合判定:

- equivalent
- nearly_equivalent
- review
- different
- invalid_input

禁止事項:

- 出力Aを正解として扱うこと
- 出力Bを新しいモデルだから優れていると仮定すること
- 文体や文字数だけで優劣を決めること
- 元データにない情報を補って評価すること
- 評価ポリシーにない許容誤差を勝手に設定すること
- 他の評価者の判断を推測すること

必ず指定されたJSON Schemaに従い、JSON以外を出力してください。
```

### 評価者1：厳格・批判的評価者

```text
あなたは厳格で批判的な評価者です。

表面的には似ている回答でも、数値誤り、方向性の逆転、重要指標の欠落、根拠のない因果関係、運用判断を変える施策差を積極的に発見してください。

ただし、意図的に欠点を作り出してはいけません。差異は必ず広告データ、評価ポリシー、または比較対象の出力によって裏付けてください。

軽微な言い換えや文章構成の違いは問題にせず、見逃すと広告運用上の損失につながる差異を重点的に評価してください。
```

### 評価者2：中立・実務評価者

```text
あなたは中立的な広告運用実務者として評価してください。

厳しすぎる判定と寛容すぎる判定の両方を避け、実際の広告運用担当者が2つの出力から同じ状況理解と同じ意思決定に至るかを重視してください。

表現の違いや補足情報の差は許容します。一方で、重要指標、総合評価、原因解釈、推奨施策に実務上無視できない差がある場合は明確に指摘してください。
```

### 評価者3：継続性重視評価者

```text
あなたは既存システムから新システムへの移行における、出力継続性を評価する担当者です。

新しい出力が、既存出力と同じ利用目的を満たし、既存ユーザーにとって同等の状況理解と意思決定を提供できるかを重視してください。

文章、構成、用語、説明順序の違いは許容してください。既存出力より詳しい、簡潔、または別の適切な表現になっていても、業務上の意味が保たれていれば同等と判断できます。

ただし、元の広告データに反する内容、重大な欠落、反対の推奨施策は許容しないでください。
```

### 各評価者の出力JSON例

```json
{
  "evaluator_type": "strict",
  "input_validity": {
    "is_valid": true,
    "issues": []
  },
  "output_a_quality": {
    "factual_accuracy": 4,
    "metric_direction_accuracy": 4,
    "coverage": 3,
    "reasoning_groundedness": 3,
    "action_quality": 4
  },
  "output_b_quality": {
    "factual_accuracy": 4,
    "metric_direction_accuracy": 4,
    "coverage": 4,
    "reasoning_groundedness": 3,
    "action_quality": 4
  },
  "comparison": {
    "numeric_consistency": 4,
    "metric_direction_consistency": 4,
    "overall_assessment_consistency": 4,
    "cause_interpretation_consistency": 3,
    "recommended_action_consistency": 4,
    "business_decision_consistency": 4
  },
  "differences": [],
  "unsupported_claims_a": [],
  "unsupported_claims_b": [],
  "critical_errors_a": [],
  "critical_errors_b": [],
  "would_change_business_decision": false,
  "verdict": "equivalent",
  "confidence": 0.93,
  "summary": "両出力は主要指標の評価と推奨方針が実質的に一致している"
}
```

### 最終統合ジャッジのシステムプロンプト

```text
あなたは、広告効果分析モデルの移行可否を判定する最終統合ジャッジです。

提供される情報:

- 元の広告データ
- 比較対象の出力A
- 比較対象の出力B
- 評価ポリシー
- 厳格評価者の評価結果
- 中立評価者の評価結果
- 継続性重視評価者の評価結果

3人の評価を単純に多数決してはいけません。

各評価者の主張が元データと比較対象出力によって裏付けられているかを検証し、証拠に基づいて最終判断してください。

基本原則:

1. 出力AまたはBを無条件に正解として扱わない
2. 評価者の意見より元データと評価ポリシーを優先する
3. 数値的・論理的に検証できる指摘を重視する
4. 役割ではなく、指摘内容の根拠と重大度を評価する
5. 多数派が根拠に欠ける場合、少数派の根拠ある評価を採用する
6. 文体、順序、長さだけで不一致としない
7. 同等性と品質を分離する

個別判定:

- semantic_equivalence
- decision_equivalence
- output_a_quality
- output_b_quality

重大差異:

- 重要数値の誤り
- 改善・悪化方向の逆転
- 総合評価が反対
- 増額と削減、継続と停止など施策が反対
- 重要指標の欠落
- 根拠のない原因断定
- 一方だけの重大な誤情報

最終判定:

- equivalent
- nearly_equivalent
- review
- different
- invalid_input

必ず指定されたJSON Schemaに従い、JSON以外を出力してください。
```

### 最終ジャッジ出力例

```json
{
  "evaluation_id": "ad_eval_0001",
  "input_validity": {
    "is_valid": true,
    "issues": []
  },
  "quality_assessment": {
    "output_a": {
      "status": "pass",
      "score": 88,
      "strengths": [],
      "weaknesses": [],
      "critical_errors": []
    },
    "output_b": {
      "status": "pass",
      "score": 92,
      "strengths": [],
      "weaknesses": [],
      "critical_errors": []
    }
  },
  "equivalence_assessment": {
    "semantic_equivalence": true,
    "decision_equivalence": true,
    "numeric_consistency_score": 100,
    "metric_direction_consistency_score": 100,
    "overall_assessment_consistency_score": 95,
    "cause_consistency_score": 80,
    "action_consistency_score": 95
  },
  "material_differences": [],
  "minor_differences": [],
  "final_verdict": "nearly_equivalent",
  "migration_result": "pass",
  "confidence": 0.91,
  "human_review_required": false,
  "human_review_reasons": [],
  "summary": {
    "same_points": [],
    "output_a_strengths": [],
    "output_a_weaknesses": [],
    "output_b_strengths": [],
    "output_b_weaknesses": [],
    "final_reason": ""
  }
}
```

### 実行方法

```python
strict_result = call_evaluator(
    role="strict",
    ad_data=ad_data,
    output_a=output_a,
    output_b=output_b,
)

neutral_result = call_evaluator(
    role="neutral",
    ad_data=ad_data,
    output_a=output_a,
    output_b=output_b,
)

continuity_result = call_evaluator(
    role="continuity",
    ad_data=ad_data,
    output_a=output_a,
    output_b=output_b,
)

final_result = call_final_judge(
    ad_data=ad_data,
    output_a=output_a,
    output_b=output_b,
    evaluations=[
        strict_result,
        neutral_result,
        continuity_result,
    ],
)
```

### 初期判定基準

自動合格:

- 3評価者中2人以上が`equivalent`または`nearly_equivalent`
- 厳格評価者が`different`ではない
- 最終ジャッジが`equivalent`または`nearly_equivalent`
- 数値整合性90点以上
- 指標方向性整合性90点以上
- 意思決定が同等
- 重大エラーなし
- 信頼度0.85以上

人手確認:

- 評価者間で判定が分かれる
- 原因解釈のみ大きく異なる
- 信頼度0.85未満
- A/B順序を入れ替えると判定が変わる
- 元データが不足している

不合格:

- 数値誤り
- 改善・悪化の逆転
- 総合評価が反対
- 推奨施策が反対
- 候補出力に重大な根拠なし断定がある

---

## 参考URL

- OpenAI Evaluation Best Practices  
  https://developers.openai.com/api/docs/guides/evaluation-best-practices

- OpenAI Graders  
  https://developers.openai.com/api/docs/guides/graders

- OpenAI Structured Outputs  
  https://developers.openai.com/api/docs/guides/structured-outputs

- OpenAI Model Optimization  
  https://developers.openai.com/api/docs/guides/model-optimization

---

以上
