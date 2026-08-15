# Repository as Memory

AIにソフトウェア開発を任せるとき、過去のチャットだけをprojectの記憶として扱うと、session変更やmodel変更のたびに前提が失われたり、会話上の説明と現在のrepositoryが食い違ったりします。

Repository as Memory は、**開発を再開するために必要な事実をrepository側へ残し、AIが交代してもprojectの現在地を確認できるようにする**という考え方です。

## What this tries to preserve

重要なのは、特定のMarkdownファイル名ではありません。

projectには、長く維持したい意図と、その時点だけの作業状況があります。また、実装されたこと、確認されたこと、まだ分からないことも同じではありません。

それらを会話の流れだけに埋め込まず、後から検証できるproject artifactsへ残すことを重視します。

## Why it matters

この考え方は、主に次のような問題を減らすためのものです。

- 新しいAI sessionが古い会話を知らず、誤った前提から作業を始める
- 古い説明を現在の実装事実だと思い込む
- 未確認事項を「完了」として扱う
- projectの制約や未解決事項がsession間で消える
- 状態が分からないのにAIが推測で作業を続ける

詳しくは以下を参照してください。

- [Design Principles](./PRINCIPLES.md)
- [Failure Modes](./FAILURE_MODES.md)
- [AI Guidance](./AGENT.md)

## Scope

この考え方は、Codex / Claude Code / ChatGPTなどを利用する個人・小規模チームのAI-assisted developmentを主な対象にしています。

本Playbookは大企業向け監査制度、ISO/SOC2等の認証・規格準拠そのものを提供するものではありません。
一方で、AI-assisted development におけるrepository authority、handoff、state management、review/evidence設計の既存プロジェクトへの導入・適用相談は受け付けています。

完全自律開発や、特定モデル専用のprompt techniqueを目的にはしていません。
