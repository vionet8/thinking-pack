# MetaのMuse（ミューズ）: 個人向けAIエージェントと、Museモデル群（2026-09 調査）

**grounding**: paraphrase・要検証。Web検索の結果（検索結果の要約）をもとにした整理。この環境からは
Meta公式ページや記事本文を開けなかった（ネットワークの制限）ため、本文は読んでいない。
数字（料金・ダウンロード数・必要メモリ・ベンチマーク）は出典ごとに食い違いがあるので、使う前に元の資料で確かめること。

## 結論

「Muse」はMetaが2026年に出した2つのものの名前。

1. **Muse（アプリ）**: 2026年9月8日に米国で始まった、個人向けのAIエージェント。質問に答えるだけでなく、
   メール・予定・予約・買い物などを本人の代わりに実行する。日本では「近く提供」と報道されているが、時期は未定。
2. **Museモデル群**: Metaの新しい研究組織（Meta Superintelligence Labs）が作るモデルの名前。
   Muse Spark（主力。APIのみ）、Muse Glimmer（手元で動かせる公開モデル）、Muse Code、Muse Image、Muse Video。

名前が同じ別物: Microsoftの「Muse」（ゲームの映像を生成する世界モデル、2025年）、
arXivの「MUSE」（未知の状況で自分の能力を判断するエージェントの研究）。

## 1. Muse（個人向けAIエージェント）

| 項目 | 内容 |
|---|---|
| 始まり | 2026年9月8日、米国。iOS・Android・Web・Mac。AIグラスにも搭載予定 |
| できること | メールの下書きと送信、予定の衝突の解消、レストラン・チケットの予約、買い物、支出の把握とサブスクの見直し、健康データからの運動・食事の計画、電話をかける。話している間も裏で作業を進める |
| つながる先 | Gmail、Googleカレンダー・Workspace、OpenTable、Ticketmaster、Spotify、Apple Health、Peloton、Plaid（銀行）、Facebook、Instagram。Walmart・GameStop・Gap・Wayfairでの買い物を追加。Amazonは遮断しているとの報道 |
| 記憶 | FacebookとInstagramから最初の人物像を作り、Gmailやカレンダーをつなぐと、1週間の過ごし方（定例の会議、集中日、サブスクの支出など）をまとめる |
| 料金 | 無料、Power（月20ドル）、Maximum（月100ドル）。無料でもカードの登録が要る。違いは機能ではなく使える量 |
| 安全の説明 | エージェントごとに専用の仮想マシン（クラウド上の専用コンピュータ）で動かし、パスワードなどは別の保管場所に置く |
| 普及 | 米国のApp Storeで無料アプリ1位、9月下旬で340万ダウンロード超（報道） |
| 気になる点 | ・Metaが決済情報やアプリへのアクセスを預かることへの信頼（Cambridge Analyticaの件）<br>・電話の一部を人間の委託先がこっそり代行する「人間のコンシェルジュ」を試験中と報道された（ロイター）。社内からも、機密情報が委託先に渡るおそれ、「専用の仮想マシンで守る」という売りを損なう、という批判 |
| 評価（CNN） | 予約・連絡・調べものを組み合わせる「計画もの」が得意。進められない理由（行き詰まり）をきちんと説明できる点が良い |

## 2. Museモデル群

| モデル | 時期 | 中身 |
|---|---|---|
| Muse Spark | 2026年4月発表、7月に1.1、8月に1.2、その後1.3 | 主力モデル。マルチモーダルの推論、コーディング、道具・コンピュータの操作。100万トークンの文脈。APIのみ（重みは未公開、将来公開の方針と報道）。料金は入力100万トークンあたり1.25ドル、出力4.25ドル（標準）。学習への利用を許すと大幅に安くなる枠もある |
| Muse Code | 2026年8月 | ターミナルで動くコーディングエージェント |
| **Muse Glimmer** | 2026年8月10日 | **30Bの公開モデル（Apache 2.0、商用利用可）**。Spark から蒸留。画像も読める。文脈128K、知識は2026年1月まで、100以上の言語で学習。4bit量子化で言語部分が20GB未満になり、**24GB〜32GBのGPU 1枚で動く**。道具の呼び出し、長い作業、失敗からの立て直しに調整。Ollama・llama.cpp・vLLMで動かせる。エージェントの評価で同じ規模のGemma 4（31B）やQwen 3.6（27B）を上回ると報じられている |
| Muse Image / Muse Video | 2026年 | 画像・動画の生成 |

## この人の関心とのつながり

- **AIを使い倒す・自律して動くAI**: Museは「答えるAI」から「代わりにやるAI」への流れの代表例。
  いまこのリポジトリでClaudeにやらせている秘書の仕事（Gmail・カレンダー・ドライブ）と、ほぼ同じ範囲を
  消費者向けアプリとしてまとめたもの。違いは、MuseはFacebook・Instagramの情報から人物像を作る点と、
  買い物・電話まで踏み込む点。日本ではまだ使えない。
- **手綱（ハーネス）**: 専用の仮想マシンで隔離する、という安全の設計と、「人間のコンシェルジュ」の報道の
  食い違いは、「任せる範囲をどう縛るか」の実例として読める。
- **仕事の閉じたネットワークで動かすAI**: 設計メモでは、外部のAPIが使えないため、手元で動かす
  モデルを候補にしていた（中国系を外し、Mistral・Llama・Gemma・Swallow・ELYZA）。**Muse Glimmer は
  米国Metaの公開モデル、Apache 2.0、GPU 1枚で動く**ので、この条件に合う新しい候補になる。
  ただし日本語の質、専門用語の扱い、社内の許可はまだ確かめていない。GPUがあるかどうかの確認が
  先に要るのは変わらない。

## 出典（検索結果）

- [Introducing Muse: The World's First Personal AI Agent Built for Everyone（Meta）](https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/)
- [Introducing Muse Spark（Meta, 2026-04）](https://about.fb.com/news/2026/04/introducing-muse-spark-meta-superintelligence-labs/)
- [Introducing Muse Spark 1.1（AI at Meta）](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/)
- [Muse Glimmer（Meta for Developers）](https://developer.meta.com/ai/models/muse-glimmer/)
- [meta-models/Muse-Glimmer-30B（Hugging Face）](https://huggingface.co/meta-models/Muse-Glimmer-30B)
- [Muse Spark（Wikipedia）](https://en.wikipedia.org/wiki/Muse_Spark)
- [Everything new coming to Meta's AI agent Muse（TechCrunch, 2026-09-23）](https://techcrunch.com/2026/09/23/everything-new-coming-to-metas-ai-agent-muse/)
- [Meta says its Muse AI agent can do things for you. I put it to the test（CNN, 2026-09-23）](https://www.cnn.com/2026/09/23/tech/meta-muse-ai-agent)
- [Meta testing a 'human concierge' for its new personal AI agent, Muse（ロイター, BNN Bloomberg掲載）](https://www.bnnbloomberg.ca/business/artificial-intelligence/2026/09/22/meta-testing-a-human-concierge-for-its-new-personal-ai-agent-muse-reuters-exclusive/)
- [Meta's Muse Arrives With Three Pricing Tiers and Payment Access（Yahoo Finance）](https://finance.yahoo.com/technology/ai/articles/metas-muse-arrives-three-pricing-183437528.html)
- [メタのAI「ミューズ」日本で開始へ 買い物代行、情報漏洩にはリスク（日本経済新聞）](https://www.nikkei.com/article/DGXZQOGN24ATS0U6A920C2000000/)
- [Metaの個人向け新AIエージェント「Muse」、機密コンピューティングに注目（日経クロステック）](https://xtech.nikkei.com/atcl/nxt/column/18/00692/091000198/)
- [Meta、ローカルAIエージェント向け30Bオープンウェイトモデル「Muse Glimmer」を発表（gihyo.jp）](https://gihyo.jp/article/2026/08/muse-glimmer)
- [メタの新モデル「Muse Glimmer」を検証（GMO天秤AIメディア）](https://tenbin.ai/media/ai_verification/meta-muse-glimmer)
- [I tried running Meta's open-weight Muse Glimmer 30B on DGX Spark（DevelopersIO）](https://dev.classmethod.jp/en/articles/dgx-spark-muse-glimmer-first-touch/)
- [Muse Spark 1.3 - API Pricing & Benchmarks（OpenRouter）](https://openrouter.ai/meta/muse-spark-1.3)
