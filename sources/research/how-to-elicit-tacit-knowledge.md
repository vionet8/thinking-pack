# 暗黙知の聞き出し方: 質問の技術と手順（2026-09 調査）

**grounding**: paraphrase・要検証。Web検索の結果（要約）と、広く知られた手法の一般的な説明をもとにした
整理。論文の本文は読んでいない。「インタビューの代役」と知識監査の問いの項目は、検索結果では
確認できておらず、一般的な知識による。

## 目的別の手法

| 目的 | 手法 | 聞き方の例 |
|---|---|---|
| どの件を話すか決める | 重要事象法（Critical Incident Technique）／重要判断法（Critical Decision Method） | 「これまでで一番対応に困った件は？」「過去最大の失敗と、そこからの教訓は？」 |
| 思い出しを助ける（場面に戻す） | 認知インタビューの「文脈の再現」 | 「そのとき何の作業中で、画面に何が出ていて、誰とやり取りしていましたか」 |
| 思い出しを助ける（当時の物を見せる） | 刺激想起（stimulated recall） | 当時のチャット・解析結果の図・設定ファイルを見せて「ここで何を考えていましたか」 |
| 判断の分かれ目を掘る | 重要判断法の深掘り | 「何を見て気づいた？」「ほかにどんな可能性を考え、なぜ外した？」「経験の浅い人ならここでどう間違える？」「もし◯◯が違ったら判断は変わる？」 |
| コツや気づきを網羅的に拾う | 応用認知タスク分析（ACTA）の知識監査 | 全体像（全体をどう捉えるか）、気づき（他の人が見落とすもの）、コツ（近道）、異常（おかしいと感じるサイン）、予測（この先どうなるか）、臨機応変（手順どおりにいかないとき）、自己点検（自分のやり方が合っているかをどう確かめるか） |
| 前例のない場面での判断を聞く | ACTAのシミュレーション面接 | 難しい仮の場面を示し、「まず何を確かめる？ 次は？」と少しずつ情報を足す |
| 判断の基準（何で見分けているか）を出す | レパートリー・グリッド | 3つの事例を並べて「2つに共通で、1つだけ違う点は？」 |
| 理由の階層を掘る | ラダリング | 「それはなぜ大事？」を繰り返す（なぜなぜ） |
| 作業中の思考をとらえる | 発話思考（think-aloud） | 作業しながら考えていることを実況してもらう（ただし作業そのものを乱すことがある） |
| 理解のずれを直す | 教え返し（teach-back） | 聞き手が理解した内容を説明し、専門家に直してもらう |
| 当たり前すぎて言わないことを出す | インタビューの代役（interview to the double） | 「明日、私があなたの代わりに入るとしたら、何に気をつければいい？」 |
| 言葉にできない動きを見る | 業務観察（シャドーイング） | 作業に同行して観察する（チャットではできない） |

## 実務の目安（要検証）

- 熟達者の考えを引き出すには、2時間×8回程度の聞き取りが必要、という実務者の見解がある。
- 研究では、複数の手法を組み合わせる（半構造化面接＋教え返し＋ラダリング＋レパートリー・グリッドなど）のが
  暗黙知と形式知の両方を引き出すのに有効とされる。
- ACTAは、認知心理学の専門家でなくても使えるよう、重要判断法を簡略化したもの（使いやすく、結果が明確）。

## チャットでのAI聞き取りに向くもの

1. **刺激想起**: 元のチャット・結果の図をそのまま見せられるので、チャットと相性が最もよい。
2. **重要判断法の深掘り**: 1回に1問で、分かれ目を順に聞く。
3. **教え返し**: AIが整理したエピソードを本人に見せて直してもらう＝確認の工程そのもの。
4. **レパートリー・グリッド**: 過去の事例が3件以上たまったら、判断の基準を出すのに使う。
5. **シミュレーション面接**: ベテランに仮の場面を出し、前例のない問題への判断の型を集める。

## 出典

- Applied Cognitive Task Analysis (ACTA): A practitioner's toolkit: https://www.academia.edu/1008791/Applied_Cognitive_Task_Analysis_ACTA_A_practitioners_toolkit_for_understanding_cognitive_task_demands
- An Easier Method for Extracting Tacit Knowledge（Commoncog）: https://commoncog.com/an-easier-method-for-extracting-tacit-knowledge/
- Lessons Learned from Customizing and Applying ACTA: https://arxiv.org/pdf/2108.05622
- Laddering: technique and tool use in knowledge acquisition: https://www.academia.edu/2821363/Laddering_technique_and_tool_use_in_knowledge_acquisition
- Use a repertory grid（Emerald）: https://www.emeraldgrouppublishing.com/how-to/observation/use-a-repertory-grid
- Eliciting tacit knowledge: video-stimulated interviewing: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6283779/
- Knowledge Elicitation Methods（WPI）: https://web.cs.wpi.edu/~jburge/thesis/kematrix.html
- What is a Stimulated Recall Interview?（ATLAS.ti）: https://atlasti.com/guides/interview-analysis-guide/stimulated-recall-interviews-research
- Cognitive Interview Technique（Simply Psychology）: https://www.simplypsychology.org/cognitive-interview.html
- 暗黙知とは（現場改善ラボ）: https://tebiki.jp/genba/useful/tacit-knowledge/
- 暗黙知明確化の方法論（技能研）: http://ginouken.com/AnmokuchiHouhou.html
