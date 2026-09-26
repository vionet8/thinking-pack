# AIで熟練者の暗黙知を聞き出し、資産化する: 先行研究と事例（2026-09 調査）

**grounding**: paraphrase・要検証。Web検索の結果（検索結果の要約）をもとにした整理で、論文や記事の
本文は読んでいない。仕事で引用する前に、元の資料で確かめること。

## 結論

「AIが対話で熟練者の判断を聞き出し、構造化して、若手が使える形にする」という考え方は、
研究でも国内企業でもすでに広く取り組まれている。ゼロから考える必要はない。
差別化できるのは、(1) 分野固有（CAEなど）の判断の型、(2) 前例のない問題への仮説立てで
効果を検証すること、(3) 業務の指標（請求係数など）と結びつけること。

## 古典的な理論（何十年も前からある型）

| 理論 | 中身 | 対応するもの |
|---|---|---|
| SECIモデル（野中郁次郎） | 暗黙知と形式知の変換: 共同化→表出化→連結化→内面化 | 聞き取り＝表出化、判断の型にまとめる＝連結化、若手が使って身につける＝内面化 |
| 事例ベース推論（Case-Based Reasoning） | 4つのR: Retrieve（似た事例を探す）→ Reuse（使う）→ Revise（直す）→ Retain（新しい事例として残す） | エピソードを貯めて検索し、結果を記録して次に活かすループそのもの。2025年にLLMエージェントと組み合わせる研究のレビューあり |
| 重要事象法（Critical Incident Technique, Flanagan 1954） | 結果に強く影響した出来事を思い出して語ってもらう | 「一番困った件は？」から入る聞き方 |
| 重要判断法（Critical Decision Method, Klein ほか） | 難しかった実例を選び、時系列→判断の分かれ目（手がかり・選択肢・根拠）→「もしも」の順に深掘りする | 熟練者の判断の型を聞き出す手順。消防指揮官・設計技術者などで実績 |
| 認知タスク分析（Cognitive Task Analysis） | 聞き出す→分析する→表現する、の3段階 | 聞き取り→整理→出力の流れ |

## 研究（2025〜2026）

- **専門家の会話からLLMが推し量った暗黙知は、専門家が自分で言葉にしたものより役に立つ**
  （"Large Language Models Explain Experts Better Than Experts Themselves"）。
  → 熟練者に「書いてもらう」より、実際のやり取り（チャット）からAIに抜き出させる方が良い可能性。
- **AIが主導する聞き取りは効率的で構造がそろうが、AIがまとめた構造は情報を取りこぼす。人と組み合わせるのが最もよい**
  （"Investigating Knowledge Elicitation Automation with Large Language Models"）。
  → 人の確認を必ず入れる根拠。
- **組織内に散らばった知識を、1人ずつ順番に対話しながらAIがつなぎ合わせる**
  （"Leveraging Large Language Models for Tacit Knowledge Discovery in Organizational Contexts"）。
- **AIによる半構造化インタビューの規模を広げる研究**（"AI Conversational Interviewing"）、
  **聞き取りの上手さを評価する環境**（"ReqElicitGym"）、**経験の知識体系と組み合わせたAI聞き手**
  （"From Chat to Interview"）など、聞き手としてのAIの研究が増えている。

## 国内企業の事例・製品

| 事例 | 中身 |
|---|---|
| NTTデータ「暗黙知伝承システム」 | 聞き取り役のAI（インタビューエージェント）が熟練者と対話して引き出し、教え役のAI（チューターエージェント）が若手の質問に答える。2026年に賞を受賞 |
| 旭鉄工（自動車部品） | 現場がSlackに投稿するだけで、生成AIが整理・分類してナレッジベースに自動で追加する。**チャットを元にする形の先行例** |
| セイコーソリューションズ「AIインタビュアー」 | 「一問一答」ではなく、次に聞くべき質問をAIが提案しながら深掘りする「プロセス型」 |
| Qast「AIナレッジインタビュー」 | AIとの会話でベテランの暗黙知を引き出す |
| デロイト トーマツ「AIインタビューエージェント」 | 社員に定期的にAIがインタビューし、スキルや経験をデータにする |
| ライオン | 熟練技術者の技術継承に生成AIを使う取り組み（2024年開始） |
| 経済産業省・NEDO「GENIAC-PRIZE」 | 暗黙知の継承をテーマに58件の応募（2026年3月に受賞企業を公表） |

## 自分の設計への示唆

1. **チャットを元にする方針は、研究（会話から抜き出す方が良い）と事例（旭鉄工）の両方で支持されている。**
2. **人の確認は外せない**（AIのまとめは取りこぼす）。
3. **「一問一答」ではなく深掘り型**にする。具体的な件や場面から入る（重要判断法の手順）。
4. **作るか、買うか**を決める。製品はすでにあるが、社内の閉じたネットワークで動かせるかは要確認。
   自前で作る価値は、分野固有の判断の型と、前例のない問題での検証にある。

## 出典

- Critical decision method for eliciting knowledge（Klein et al.）: https://ieeexplore.ieee.org/document/31053/
- Use of the Critical Decision Method to Elicit Expert Knowledge（Hoffman et al., 1998）: https://journals.sagepub.com/doi/10.1518/001872098779480442
- Critical incident technique: https://en.wikipedia.org/wiki/Critical_incident_technique
- Cognitive Task Analysis: Eliciting Expert Cognition in Context（2025）: https://journals.sagepub.com/doi/10.1177/10944281241271216
- Review of Case-Based Reasoning for LLM Agents: https://arxiv.org/pdf/2504.06943
- Large Language Models Explain Experts Better Than Experts Themselves: https://arxiv.org/pdf/2608.07488
- Investigating Knowledge Elicitation Automation with Large Language Models: https://www.semantic-web-journal.net/system/files/swj3868.pdf
- Leveraging LLMs for Tacit Knowledge Discovery in Organizational Contexts: https://pith.science/paper/2507.03811
- AI Conversational Interviewing: https://arxiv.org/pdf/2606.20064
- ReqElicitGym: https://arxiv.org/html/2602.18306
- From Chat to Interview: https://arxiv.org/pdf/2605.05828
- NTTデータ 暗黙知伝承（ユーザー変革賞）: https://www.nttdata.com/jp/ja/trends/data-insight/2026/0520/
- NTTデータ AIエージェントがインタビュアーに: https://www.nttdata.com/jp/ja/trends/data-insight/2026/0209/
- 日本ユーザー協会（旭鉄工の事例を含む）: https://www.jtua.or.jp/ict/solution/reform/knowledge-inheritance/202504_01/
- セイコーソリューションズ AIインタビュアー: https://www.seiko-sol.co.jp/archives/92152/
- Qast AIナレッジインタビュー: https://aismiley.co.jp/ai_news/any-qast-ai-interview/
- デロイト トーマツ AIインタビューエージェント: https://www.deloitte.com/jp/ja/about/press-room/nr20250213.html
- ライオン 生成AIによる暗黙知伝承: https://doc.lion.co.jp/uploads/tmg_block_page_image/file/9885/20240603_02.pdf
