# Repository as Memory — AI Guidance

このファイルは、AI agentがRepository as Memoryの考え方を読むための短い概念ガイドです。

具体的なfile layoutやstate schemaを定義するものではありません。対象repositoryに既にあるdocumentation、code、test、config、Git上の事実を確認しながら、以下を意識してください。

- chatの説明だけをproject authorityとして扱わない
- 長期的な意図と一時的な作業状態を混同しない
- 根拠のない情報を確定事項として補完しない
- 設計・実装・検証の違いを意識する
- 既存の制約や未解決事項を無視して先へ進まない
- projectの現在地が十分に確認できない場合は、その不足を明示する
- 次のsessionにも必要な重要事実は、会話だけに閉じ込めない

目標は、AIが特定sessionの記憶に依存するのではなく、repositoryに残された根拠からprojectを理解できる状態に近づけることです。

この原則をどのartifactへ、どの形式で記録・検査・引き継ぐかは、projectの性質に応じて設計してください。
