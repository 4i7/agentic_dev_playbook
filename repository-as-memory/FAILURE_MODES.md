# Failure Modes

Repository as Memory が対象にするのは、AIそのものの性能不足だけではありません。

長期開発では、modelが十分に高性能でもproject側の状態管理が曖昧だと、次のような失敗が起こります。

## Session drift

新しいsessionが以前の判断や現在地を正確に引き継げず、似ているが別の前提で作業を続けます。

## Stale narrative

過去の説明が残り続け、現在のcodeや設定より古い情報が正しいものとして扱われます。

## False completion

設計、実装、test、実利用での確認など、成熟度の異なる状態が一つの「完了」に圧縮されます。

## Lost constraints

一時的に会話で決めた制約や保留事項が次のsessionへ伝わらず、避けるべき変更が再び提案されます。

## Confident guessing

不足しているproject stateをAIが自然な推測で補い、その推測を前提に変更を重ねます。

## Handoff without evidence

「引き継げるはず」という期待だけがあり、新しいsessionが実際に何を復元できるのか確認されていません。

Repository as Memory は、これらを完全に消すことを保証するものではありません。重要なproject factsを会話から切り離し、後から確認可能にすることで、失敗を検出・停止しやすくするための考え方です。
