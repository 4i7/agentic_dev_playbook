# Design Principles

Repository as Memory は、特定のtoolやfile naming conventionではなく、project continuityのための設計原則です。

## 1. Durable facts over conversational memory

継続に必要な事実は、特定sessionの会話だけに依存させません。

AIが交代しても確認できる場所に、projectの根拠を残します。

## 2. Separate what is stable from what is temporary

長期的な設計意図と、その時点の作業状況は別物として扱います。

同じ情報を何度も複製するのではなく、性質の違う情報が混ざらないことを重視します。

## 3. Evidence and intention are different

「こうしたい」という設計と、「実際にそうなっている」という事実は分けます。

testや実環境で確認されていないものを、説明だけで成立済みと扱いません。

## 4. Preserve uncertainty

分からないことを、AIの推測で埋めません。

未確定であること自体をprojectの状態として残せる方が、長期的には安全です。

## 5. Make continuation inspectable

次の担当者やAIが、以前の会話を知らなくても「何を根拠に続けるべきか」を確認できる状態を目指します。

## 6. Stop when the basis for continuation is unclear

現在の状態や根拠を十分に確認できない場合、推測で先へ進むより、確認不足を明示することを優先します。

これらの原則を、どのファイルへ、どの形式で、どこまで機械化するかはprojectごとに異なります。
