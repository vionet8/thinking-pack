# 複数のCAEソフトを1つのきっかけで連続実行する（ワークフローの統括）（2026-10-20）

**grounding**: 本人が貼った、別のAIチャットの回答を Claude が整理したもの（paraphrase・要検証）。
各ソフトをGUIなしで動かせるかは、検索とClaudeの知識で確かめた（下の「確かめたこと」）。
業務の分野や社内ツールの名前など、仕事の特定につながる部分は一般化した。

## このメモを使う場面

- **使う条件**: 複数の解析ソフトにまたがる手作業の流れ（前処理→計算→後処理→レポート）を、自動でつなぎたいとき。
  例（これに限らない）: 定型の構造解析・熱解析、条件を変えた比較計算、納品前のレポートづくり、参考モデルの設定の読み取り。
- **向かない場面**: 1回きりの探索的な解析、毎回GUIで人が判断する工程、ライセンスや実行環境が自動実行を許さない場合、
  繰り返す回数が少なく自動化の手間が見合わない場合。

## 要点

1. **ソフトを1つに統合しない。** 各ソフトが持つ機能を外の制御プログラムから呼び出し、結果を次の工程に渡す
   （ワークフローの統括＝オーケストレーション）。単一ソフトの操作の自動化とは考え方が違う。
2. **流れの例**:

   | 順 | ソフト（例） | やること |
   |---|---|---|
   | ① | きっかけ | 案件・解析条件・入力ファイルを指定して開始 |
   | ② | ANSA | モデル読み込み、形状処理、メッシュ作成、チェック |
   | ③ | Abaqus | 入力データ生成、解析実行、結果ファイルの出力 |
   | ④ | HyperView | 結果読み込み、コンター表示、画像・データの出力 |
   | ⑤ | Python など | 数値の集計、グラフ、条件どうしの比較 |
   | ⑥ | レポート生成 | HTML・PDF・画像・解析ログをまとめて出力 |

3. **中心は各ソフトではなく、全体の順番と状態を管理する制御プログラム。**

   | 管理すること | 役割 |
   |---|---|
   | 起動 | 各ソフトの起動コマンドと引数 |
   | 工程 | 前の工程が終わってから次を実行 |
   | データの受け渡し | ファイルや解析条件を次の工程へ |
   | エラー | 失敗したら止め、原因とログを残す |
   | 結果の確かめ | 計算が終わったかだけでなく、結果ファイルがあるか・中身が期待どおりか |
   | レポート | 画像・数値・条件・ログを集める |

4. **一番大事な区別: ソフトのプロセスが終わったことと、解析が正常に終わったことは別。**
   計算の完了を確かめ、期待する結果が出ていることを確かめてから、次の工程に渡す。
5. **GUIは要る所だけ。** バッチで動く工程は裏で処理し、画面が要る工程だけGUIを起動する。
6. **AIを組み込むなら、承認済みの処理を部品にして、AIは組み合わせと条件指定だけを担当する。**
   AIに自由にスクリプトを書かせて実行させるより、安全で、同じ結果を再現しやすい。
   例（これに限らない）: 「この条件で熱解析を流して、前回条件との差と温度分布をまとめて」と頼むと、対応する部品の組み合わせが実行される。
7. **始め方: 解析1本で、起動からレポートまでを全部自動で通す最小構成を作る。** その後、共通の制御基盤を残したまま、
   解析の種類やソフトを増やす。独自の道具（例: 参考モデルのレポートツール）を作るときも、後でこの基盤につなぐ前提で設計しておく。

## 確かめたこと

| 主張 | 確認の結果 |
|---|---|
| Abaqus はGUIなしでPythonスクリプトを動かせる | Claudeの知識（`abaqus cae noGUI=スクリプト.py` の形）。検索はしていない |
| HyperView（HyperWorks）はスクリプトのバッチ実行とレポート生成ができる | 確認。`hw -b -tcl スクリプト.tcl` の形。公式チュートリアルに、テンプレートからレポートをバッチで作る例がある。**注意**: バッチでの画面キャプチャは小さくなることがあり、ハードやドライバに左右される（大きさを指定する命令を使う）。入力待ちの画面を出す処理はバッチでは動かない |
| ANSA はGUIなしで動かせる | 確認。`ansa -nogui -i モデル -exec 関数()` の形。BETA CAE の講習にバッチメッシュの単元がある。正確なオプションは版ごとに `ansa -help` で確かめる |
| どこまで自動化できるかは、版・ライセンス・API・社内の実行環境で変わる | そのとおり。特にライセンス（同時に使える数、バッチ実行の扱い）は最初に確かめる（Claudeの見立て） |

## GUIを開く工程でも、起動からスクリプトの実行まで自動にできるか（2026-10-20 追記）

**結論: 3つともできる。** 画面を開いたままでも、起動と同時にスクリプトを走らせる方法がある。上から順に試す。

1. **起動の命令にスクリプトを付ける**（一番確実。工程ごとに違うスクリプトを渡せる）
2. **起動時に必ず読まれるファイルに書く**（毎回同じ準備をさせたいとき。例: 自作の関数やボタンを読み込む）
3. **画面操作の自動化**（1・2が使えないときの最後の手段）

### ソフトごとのやり方

| ソフト | 1. 起動の命令に付ける（画面は開いたまま） | 2. 起動時に読まれるファイル | 参考: 画面なし | 確認 |
|---|---|---|---|---|
| Abaqus/CAE | `abaqus cae script=スクリプト.py`（Viewerは `abaqus viewer script=…`）。環境ファイルの設定の後に実行される | 環境ファイル（各自はホームの `abaqus_v6.env`、全員分はインストール先の `custom_v6.env`）に `def onCaeStartup():` を書く。中身はマクロマネージャで記録した命令を貼ってもよい | `abaqus cae noGUI=スクリプト.py`（終わると閉じる） | 公式文書で確認。`noGUI` では動かないスクリプトが `script=` では動いた、という報告がある（画面がある前提の命令が混ざっていると起きる） |
| HyperMesh（従来の画面、Tcl） | `hmopengl -tcl<ファイル>`（`-tcl` とファイル名の間に空白を入れない） | `hmcustom.tcl`（`HW_CONFIG_PATH` → ホーム → 起動フォルダの順に探す。`-tcl` の方が先に実行される） | `-batch`（版によって `-b`）を付けると、スクリプトの後に閉じる | 公式文書（2017・2019版）で確認 |
| HyperMesh（新しい画面、Python） | `runhwx -startwith HyperMesh -f スクリプト.py` | 未確認（画面の中では IPython の `run ファイル` で実行できる） | `-b -f` は「何も起きない」という報告がある | コミュニティの回答のみ。公式文書では未確認 |
| HyperView | `hw -tcl スクリプト.tcl`（`-b` を付けなければ画面は開いたまま。新しい版は `runhwx`）。HWCスクリプトは `-hwc` | 未調査 | `hw -b -tcl スクリプト.tcl` | 公式チュートリアル・コミュニティで確認 |
| ANSA | `ansa -i モデル -exec "関数()"`（モデルを開いて、起動後に関数を呼ぶ）。`-execscript スクリプト.py` / `-execpy "命令"` もあるはず | `ANSA_TRANSL.py`（ホームの版ごとのフォルダ。全員分ならインストール先の config）に `ansa.ImportCode('スクリプト.py')` を書く。起動時に読み込まれ、関数が実行画面に並ぶ | `-nogui` | `-i`・`-exec`・`-nogui` は確認。`-execscript`・`-execpy` はClaudeの知識で未確認。`ANSA_TRANSL` は2013年のフォーラム（ANSA 14）と講習の目次で確認。今の版は `ansa -help` と手引きで確かめる |

どのソフトも、版によって命令の名前が変わる。最初に、使っている版の起動オプションの一覧（`-help` や手引きの「起動オプション」の頁）で確かめる。

### スクリプトの作り方（ゼロから書かない）

- 手作業を記録して、それを直す。Abaqus/CAE は操作が記録ファイル（`.rpy`）とマクロに残る。HyperMesh は操作が命令ファイル（`command.tcl` など）に残る（Claudeの知識）。
- 記録をそのまま使うと、画面上の選び方（クリックした位置や番号）が入り込んで、別のモデルで動かないことがある。名前や条件で選ぶ形に直す。

### 3. 画面操作の自動化（最後の手段）

- メニューやボタンを外から押す道具（Windowsなら pywinauto、AutoHotkey など）。
- 壊れやすい。画面の配置・解像度・版上げ・言語設定が変わると動かない。確認の画面が1つ増えるだけで止まる。
- 使うとしても「起動してスクリプトを読み込むところ」までにして、中身の処理は1・2のスクリプトでやる。

### 画面を開いたまま自動で動かすときの注意（制御プログラムとの受け渡し）

1. **終わったことの知らせ方を決める。** 画面が開いたままなので、プロセスの終了では判断できない。スクリプトの最後に
   「完了ファイル」（状態・出力したファイル・エラーを書いたもの）を書き、制御プログラムはそれを待つ。
   本文の「プロセスの終了 ≠ 正常な完了」と同じ考え方。
2. **閉じるかを決める。** 次の工程に自動で進むなら、スクリプトの最後でソフトを閉じる。人が画面を見て確かめる工程なら、開いたままにして、人の確認を合図に次へ進む。
3. **待ち時間の上限を決める。** ライセンス待ちや、確認の画面で止まったときに備えて、上限を過ぎたら止めて知らせる。
4. **入力待ちの画面を出さない。** 上書きの確認などが出ると止まる。スクリプトの中で上書きの指定をしておく。
5. **ログインした画面の中で動かす。** 画面のあるソフトは、ログインしていない状態（サービスや「ログオフ中も実行」のタスク）から起動すると、画面が出ない・失敗することがある（Claudeの見立て）。
6. **画像の大きさを指定する。** 画面の大きさで出力画像が変わる。HyperViewではバッチで画像が小さくなる報告がある。大きさを指定する命令を使う。

### 最初の試し（例。これに限らない）

1ソフト・スクリプト1本で、「起動 → スクリプト実行 → 完了ファイルを書く → 閉じる」を通す。
例: Abaqus/CAE を `script=` で起動し、モデルを開いて部品の数を数え、結果をファイルに書いて閉じる。
通ったら、制御プログラムから起動して、完了ファイルを待って次の工程に進む形にする。

## このプロジェクトとの接点

- **参考モデルレポートツールの設計メモ**（`cae-reference-model-report-tool.md`）: レポートツールの出力が、この流れの「結果の確かめ」「レポート」の部品になる。
  目印の基準「影響の大きさ × 気づきにくさ」は、自動の結果確認で何を確かめるかの選び方にもそのまま使える。
  単位系のずれや結合の切れは、計算は最後まで終わるのに結果がおかしい典型で、「プロセスの終了 ≠ 正常な完了」の確認対象の筆頭。
- **AIエージェントの7つの要点**（`sources/videos/misc-videos.md`）: ツールと権限を厳密に決める、迷ったら人に委ねる、単純から始めて広げる、と同じ設計。
- **チーム共通のスキル**（`prompts/ai-stumble-loop.md`）: 「承認済みの処理を部品にして、AIが組み合わせる」は、チームで確かめたスキルを目次に並べてAIが選ぶ形と同じ。
- **AI活用を始める30分**（`thoughts/2026-10-13_ai-adoption-starts-with-30-minutes.md`）: 30分の題材にするなら、流れ全体ではなく1工程だけ
  （例: 結果の数値の集計とグラフ化）から始めると、30分で「試して障壁を知る」まで届く。

## 間違っているとしたら、ここ

- 「1つのきっかけで全部」は、途中で人の判断が要る工程（メッシュの品質、結果の妥当性）を見落としやすい。止めて人に見せる点を先に決める。
- 作る手間・直す手間・ソフトの版上げのたびに直す手間が、手作業の手間を上回ることがある。繰り返し回数で見合うかを先に見積もる。

## 出典（検索結果）

- [Starting Abaqus/CAE（起動オプション、Abaqus 2017 文書）](https://abaqus-docs.mit.edu/2017/English/SIMACAECAERefMap/simacae-c-gststart.htm)
- [Using Abaqus Scripting Interface commands in your environment file（onCaeStartup）](https://abaqus-docs.mit.edu/2017/English/SIMACAECMDRefMap/simacmd-c-intaclenvironment.htm)
- [Change settings on Abaqus/CAE startup using the environment file（TECHNIA）](https://simulation-blog.technia.com/simulation/change-settings-on-abaqus-cae-startup-using-the-environment-file)
- [noGUI と script= の違いの報告（iMechanica）](https://imechanica.org/node/12750)
- [HyperMesh Startup Options（Altair 2019）](https://2019.help.altair.com/2019/hwd/topics/getting_started/start_options_hypermesh_r.htm)
- [hmcustom.tcl（Altair）](https://help.altair.com/hwdesktop/hwx/topics/getting_started/hmcustomtcl_r.htm)
- [How to execute a script when opening HyperMesh（Altair Community）](https://community.altair.com/discussion/6063/how-to-execute-a-scripts-when-open-hypermesh)
- [How to run HyperMesh in batch mode（Python、-startwith と -f。Altair Community）](https://community.altair.com/discussion/62393/how-to-run-hypermesh-in-batch-mode)
- [HyperWorks Python API: Getting Started（Altair 2024）](https://2024.help.altair.com/2024/hwdesktop/pythonapi/getting_started.html)
- [HyperWorks Desktop Startup Options（Altair 2021）](https://2021.help.altair.com/2021/hwdesktop/hwd/topics/getting_started/start_options_hyperworks_desktop_r.htm)
- [ANSA script（ANSA_TRANSL.py の置き場所と書き方。CFD Online）](https://www.cfd-online.com/Forums/ansa/111581-ansa-script.html)
- [ANSA scripting basic（BETA CAE 講習の目次）](https://www.beta-cae.com/courses/ansa_scripting_basic.pdf)

- [Create Reports in Batch Mode（Altair HyperWorks Desktop）](https://help.altair.com/hwdesktop/hwx/topics/reportsX/tut/tut_hwd_0092_t.htm)
- [Batch Mode from HyperView（Altair Community）](https://community.altair.com/discussion/25034/batch-mode-from-hyperview)
- [How to run a script for HyperView/HyperGraph from a command line（Altair Community）](https://community.altair.com/discussion/35127/how-to-run-a-script-for-hyperview-hypergraph-from-a-command-line)
- [Configuring the Execution Options（ANSA、Isight連携の説明）](https://abaqus.uclouvain.be/English/IhrComponentMap/ihr-t-ANSA-Execution-Options.htm)
- [Scripting with ANSA（BETA CAE）](https://beta-cae.us/?p=3794)
