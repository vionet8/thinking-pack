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

## Abaqus/CAE: .cae と .jnl を開き、中のジョブを流す（コア数と作業フォルダつき）（2026-10-20 追記）

本人の今の手作業: .cae と .jnl を開く → 作業フォルダを作る → 計算コア数を決める → 中にあるジョブを流す。これを自動にする。

### 前提: .cae と .jnl の役割

- **.cae** はモデルの本体。普段はこれを開けばよい（スクリプトでは `openMdb`）。
- **.jnl** は、保存したモデルを作り直すための操作の記録。保存のたびに追記される。
  .cae を移したりコピーしたりするときは、.jnl も同じフォルダに一緒に置く。
- **.cae が壊れた・開けないとき**だけ、.jnl から作り直す: `abaqus cae recover=名前.jnl`（拡張子を省くと .jnl とみなされる）。

### 流れ

1. **作業フォルダを作る。** 案件・条件・日付などで名前を付ける。元の .cae と .jnl を**一緒に**コピーして、元は触らない。
2. **作業フォルダに移る。** ジョブの出力（.odb・.sta・.msg など）は、今いるフォルダに出る。画面の「作業ディレクトリの設定」にあたるのは、スクリプトでは `os.chdir(作業フォルダ)`。
3. **.cae を開く。**
4. **中のジョブにコア数を設定する。** `numCpus`（使うコア数）と `numDomains`（領域の分割数。Explicit ではコア数の倍数にする必要がある）。同じ値にしておくのが無難。
5. **ジョブを流して、終わるまで待つ。** 1つずつ待つと順番に流れる。全部投げてから待つと並行して流れる。
6. **正常に終わったかを確かめて、完了ファイルを書く。**

### スクリプト例（未実行。この環境にAbaqusがないため、動作は確かめていない）

```python
# -*- coding: utf-8 -*-
# run_cae_jobs.py
# 画面なし: abaqus cae noGUI=run_cae_jobs.py -- 元の.caeのパス 作業フォルダ コア数
# 画面あり: abaqus cae script=run_cae_jobs.py -- 元の.caeのパス 作業フォルダ コア数
# （Abaqus 2023 までは Python 2.7、2024 からは Python 3。両方で動く書き方にしている）
import os, sys, shutil, json
from abaqus import *
from abaqusConstants import *

src_cae, work_dir, ncpu = sys.argv[-3], sys.argv[-2], int(sys.argv[-1])  # 引数は後ろから数える
src_dir = os.path.dirname(os.path.abspath(src_cae))
name = os.path.splitext(os.path.basename(src_cae))[0]

# 1. 作業フォルダを作り、.cae と .jnl を一緒にコピーする（元は触らない）
if not os.path.isdir(work_dir):
    os.makedirs(work_dir)
for ext in ('.cae', '.jnl'):
    src = os.path.join(src_dir, name + ext)
    if os.path.exists(src):
        shutil.copy2(src, work_dir)

# 2. 作業フォルダに移る（ジョブの出力はここに出る）
os.chdir(work_dir)

# 3. .cae を開く
openMdb(pathName=name + '.cae')

# 4・5. 中のジョブすべてにコア数を設定して、1つずつ流す
results = {}
for job_name in mdb.jobs.keys():
    job = mdb.jobs[job_name]
    job.setValues(numCpus=ncpu, numDomains=ncpu)
    job.submit(consistencyChecking=OFF)
    job.waitForCompletion()
    results[job_name] = str(job.status)  # COMPLETED / ABORTED など

# 6. 完了ファイルを書く（制御プログラムはこれを待つ）
with open('done.json', 'w') as f:
    json.dump({'cae': name, 'cpus': ncpu, 'jobs': results}, f, indent=2)
```

制御プログラム（Windows の例）から呼ぶとき:

```
cd /d D:\work\案件A_条件1_20261020
abaqus cae noGUI=C:\scripts\run_cae_jobs.py -- D:\models\案件A.cae D:\work\案件A_条件1_20261020 8
```

### 別の作り方: CAE は入力ファイルを書くだけにし、計算はコマンドで流す

```
abaqus cae noGUI=write_inp.py -- 案件A.cae      （中で mdb.jobs[名前].writeInput() を呼ぶ）
abaqus job=Job-1 input=Job-1.inp cpus=8 interactive
```

- 自動化にはこちらの方が扱いやすい（Claudeの見立て）。計算の命令にコア数（`cpus=`）を直接渡せる。終了コードと出力ファイルで結果を確かめやすい。
  CAE を閉じてから計算するので、CAE のライセンスを計算中に持ち続けない。別のマシンや計算待ちの列にも回しやすい。
- 人の手順（CAE の中でジョブを流す）に近いのは、上のスクリプト例の方。

### 正常に終わったかの確かめ方（プロセスの終了 ≠ 正常な完了）

- スクリプトの中では、待った後のジョブの状態（`job.status`）が COMPLETED か（Claudeの知識）。
- ファイルでは、`.sta` の最後に「THE ANALYSIS HAS COMPLETED SUCCESSFULLY」があるか（Standard）、`.log` にエラーで終わった記録がないか、`.odb` ができているか（Claudeの知識）。
- 計算が最後まで終わっても結果がおかしいこと（単位系のずれ、結合の切れ）は、ここでは分からない。参考モデルレポートツールの確認項目で見る。

### 注意

- **コア数とライセンス**: Abaqus はコア数が増えるほど、使うライセンス（トークン）が増える（目安: 5 × コア数^0.422 の切り捨て。8コアで約12、16コアで約16。Claudeの知識）。
  空いている数を超えるとライセンス待ちで止まるので、コア数はライセンスの空きと一緒に決める。
- **古い版で保存された .cae**: 開く前に版を上げる必要があることがある（`upgradeMdb`。Claudeの知識）。画面ありなら確認の画面が出て止まる。
- **画面ありで流すとき**: ジョブが終わるまで画面は操作できない。終わったら閉じるなら、制御プログラムは done.json を見てから次に進む。
- **ジョブが複数のモデルにまたがるとき**: `mdb.jobs` は全モデルのジョブを含む。流すジョブを名前で絞りたいなら、引数で渡す。

## 計算と後処理（CSV出力）を並行して進める（2026-10-22 追記）

**grounding**: 本人が貼った別のAIの回答（本人の考えている構成への答え）と、Claudeが検索で確かめて足したこと。

### 回答の要点

1. **「Abaqusの画面が開いていること」と「ジョブが計算中であること」は別に管理できる。** だから、1つ目のAbaqusで次のジョブを進めながら、
   別のプロセスで計算済みのジョブのCSVを出すことができる。同じモデル・同じファイルを複数のプロセスから同時に触るときは、競合を避ける設計が要る。
2. **待つのは `waitForCompletion()`。** ジョブを投入した後に呼ぶと、計算が終わるまでスクリプトがそこで止まる。自分で待つループを書かなくてよい。
   ただし、戻ってきた＝正常に終わった、ではない。`job.status` を確かめて、正常なときだけCSV出力に進む。
3. **本人の考えている構成**:
   - プロセスA（解析を管理する Abaqus/CAE）: 指定フォルダのモデルを読む → ジョブの設定を直す → Job 1 を投入して待つ → 正常終了を確かめる → Job 2 を投入する。
   - プロセスB（別のプロセス）: Job 1 の完了を確かめる → ODB を読んで CSV を出す → 終わる。
   - Aが `waitForCompletion()` で待っている間、Aのスクリプトは先に進まない。それでもBは独立して起動できるので、並行して動かせる。
4. **最初はもっと単純に作る。** 制御スクリプト（フォルダ指定・投入・待つ・次のジョブへ）と、後処理スクリプト（ODB から必要な値を CSV へ）の2本にする。
   計算と後処理を並行させたくなったら、そこで別プロセスにする。後から HyperView の画像出力やレポートを足しても、全体を作り直さずに済む。

### Claudeが確かめて足したこと

1. **後処理に Abaqus/CAE を起動しなくてよい可能性が高い。** ODB の読み取りは `abaqus python export_csv.py`（`odbAccess` を使う）でできる。
   「ライセンスのトークンを使わない」という報告が複数ある。使っている版で動かなければ、`abaqus viewer noGUI=export_csv.py`（こちらはライセンスを使う）。
   最初に社内の版で一度試して確かめる。
2. **並行は、Aの中で後処理を「起動だけして待たない」ようにすれば作れる。** Job N が正常に終わったら、後処理を別プロセスで起動し、
   終わりを待たずに Job N+1 を投入する。Job N+1 の計算中に Job N の CSV が出る。別の制御プログラムは要らない。
3. **競合を避ける決まり**:
   - モデル（.cae）を開いて変えるのは A だけ。B はモデルを開かず、ODB だけを読む。
   - ODB は計算が終わってから読む。計算中は ODB を書いている印（`.lck` ファイル）がある。計算が終わっても `.lck` が残っていたら、異常終了の疑い。
   - ODB は読み取り専用で開く。
   - ジョブごとに出力の場所（作業フォルダかファイル名）を分ける。
   - CSV は一時的な名前で書き、書き終わってから正式な名前に変える。制御プログラムが書きかけの CSV を読まないようにする。
4. **コアの取り合い**: 計算（Job N+1）と後処理（Job N）が同時に動くので、計算のコア数はマシンのコアを使い切らない（1つ残す）（Claudeの見立て）。

### スクリプト例（未実行。この環境に Abaqus がないため、動作は確かめていない）

制御側（前の `run_cae_jobs.py` の4・5の部分を置き換える）:

```python
import subprocess
EXPORT = r'C:\scripts\export_csv.py'  # 後処理スクリプトの場所

procs = []
for job_name in mdb.jobs.keys():
    job = mdb.jobs[job_name]
    job.setValues(numCpus=ncpu, numDomains=ncpu)
    job.submit(consistencyChecking=OFF)
    job.waitForCompletion()
    results[job_name] = str(job.status)
    if job.status == COMPLETED:
        # 後処理は起動だけして待たない → すぐ次のジョブへ
        cmd = 'abaqus python "%s" "%s.odb" "%s.csv"' % (EXPORT, job_name, job_name)
        procs.append(subprocess.Popen(cmd, shell=True))

for p in procs:
    p.wait()  # 最後に、後処理が全部終わるのを待ってから done.json を書く
```

後処理側:

```python
# -*- coding: utf-8 -*-
# export_csv.py  使い方: abaqus python export_csv.py 結果.odb 出力.csv
# 最後のステップの最後のフレームから、最大ミーゼス応力と最大変位を1行のCSVに書く。
# 取り出す値（出力の種類・部品・ステップ）は、解析の種類に合わせて変える。
import os, sys
from odbAccess import openOdb

odb_path, out_csv = sys.argv[1], sys.argv[2]
odb = openOdb(path=odb_path, readOnly=True)  # 読み取り専用で開く
step_name = list(odb.steps.keys())[-1]
frame = odb.steps[step_name].frames[-1]
max_mises = max(v.mises for v in frame.fieldOutputs['S'].values)
max_disp = max(v.magnitude for v in frame.fieldOutputs['U'].values)
odb.close()

tmp = out_csv + '.tmp'
f = open(tmp, 'w')
f.write('odb,step,max_mises,max_disp\n')
f.write('%s,%s,%g,%g\n' % (os.path.basename(odb_path), step_name, max_mises, max_disp))
f.close()
if os.path.exists(out_csv):
    os.remove(out_csv)
os.rename(tmp, out_csv)  # 書き終わってから正式な名前にする
```

- 大きいモデルでは、値を1つずつ回すと遅い。その場合はまとめて取り出す方法（`bulkDataBlocks`）を使う（Claudeの知識）。

### 間違っているとしたら、ここ

- `abaqus python` で `odbAccess` が使えるか、ライセンスを使わないかは、版と社内のライセンスの設定で変わりうる。最初に1回試す。
- Abaqus の中から別の Abaqus を起動すると、環境の引き継ぎのせいでうまく動かないことがある（Claudeの見立て）。そのときは、Abaqus の外の制御プログラムから後処理を起動する。

## チェック担当者が追えるログを残す（2026-10-25 追記）

**grounding**: 本人が貼った別のAIの回答と、Claudeが足したこと（「Claudeが足したこと」の節）。

### 回答の要点

- 処理を動かすためのログだけでなく、チェック担当者が「何を自動で実行し、どの条件で、どんな結果になったか」を追えるログを残す。
- CAEでは「解析完了」だけでは足りない。正常に終わっていても、設定値や入力ファイルが意図と違うことがある。
- **ログは3種類に分ける。**

  | ログ | 記録すること | 主な用途 |
  |---|---|---|
  | 操作ログ | 時刻、操作の内容、対象のモデル、設定の変更 | 自動処理の追跡 |
  | 実行ログ | ジョブの投入、開始・終了の時刻、終了状態、エラー | トラブルの調査 |
  | 検証ログ | 期待値、実際の値、チェックの結果、警告 | 解析結果の確認 |

- **特に残す4つの証跡**:
  1. 入力条件: 使ったファイル、解析条件、設定を変えた前後の値。
  2. 実行: 各処理の開始・終了の時刻、ジョブ名、終了コード、エラー。
  3. 出力: 作った CSV・ODB・画像のファイル名、件数、必要ならファイルのハッシュ値。
  4. 検証: チェック項目、期待値、実際の値、合否、警告。
- 同じ解析を後日やり直せるように、スクリプトの版と実行の設定も残す。
- **ログが残っていることと、解析が正しいことは別。** 温度のCSVを正常に出せても、抽出したステップや節点が違えば成果物は誤り。
  だから「CSV出力成功」だけでなく、どのステップ・どの領域・どの変数を抽出したかも残す。
- 人が読む形（HTMLのレポート）と、プログラムで集計する形（JSON Lines：1行に1件のJSON）を分ける。
- 最初に作るのは「実行履歴の一覧」（成功・失敗・警告がひと目で分かる）と「1回ごとの詳細レポート」（入力条件・操作の履歴・結果の検証）。

  チェック担当者が見る詳細レポートの例（表示の例で、実際の結果ではない）:

  ```
  解析実行記録　正常終了　　案件: THERMAL_001 / 実行ID: RUN_20261024_001
  15:00:02 入力ファイル読み込み　model.inp / 読み込み成功
  15:00:15 ジョブ設定変更　CPU数: 8、解析ステップ: Step-1
  15:00:20 計算開始　Job-001 を投入
  15:18:42 計算完了　正常終了を確認。経過時間 18分22秒
  15:19:05 CSV出力　ODBから温度・変位を抽出。出力ファイルを確認
  15:19:10 検証　出力件数、必須項目、値の範囲をチェック
  ```

### Claudeが足したこと

1. **入力条件の証跡は、参考モデルレポートツールの出力をそのまま使える。** あのツールが出す「モデルの設定の要約」（単位系、材料、結合、荷重・拘束など）を
   実行の前に1回出して残せば、「どんな設定のモデルを流したか」の証跡になる。目印の基準「影響の大きさ × 気づきにくさ」で、検証ログで確かめる項目も選べる
   （`cae-reference-model-report-tool.md`）。
2. **前に書いたスクリプト例とつなぐ場所**:
   - `done.json`（完了ファイル）は、実行ログの最後の1行にあたる。完了ファイルとは別に、途中の1件ごとを JSON Lines に追記していく。
   - `export_csv.py` は、CSVの中か横に「どのステップ・どのフレーム・どの変数・どの領域から取ったか」を書く（いまは最後のステップの最後のフレームの全体から取っている）。
   - 設定の前後の値は、`setValues` の前に今の値を読んで残す（例: 変える前の `numCpus`）。
3. **ログを書く小さな関数の例**（未実行。Abaqus 2023 までの Python 2.7 と、2024 からの Python 3 の両方で動く書き方）:

   ```python
   import json, time, hashlib

   def log(path, kind, **fields):
       """kind は 'operation' / 'execution' / 'verification' のどれか。1行に1件のJSONで追記する。"""
       rec = {'time': time.strftime('%Y-%m-%dT%H:%M:%S'), 'kind': kind}
       rec.update(fields)
       f = open(path, 'a')
       f.write(json.dumps(rec) + '\n')
       f.close()

   def sha256(path):
       """ファイルのハッシュ値。同じ入力ファイルを使ったかを後から確かめるため。"""
       h = hashlib.sha256()
       f = open(path, 'rb')
       for chunk in iter(lambda: f.read(1 << 20), b''):
           h.update(chunk)
       f.close()
       return h.hexdigest()

   # 使い方の例
   # log('run.jsonl', 'operation', action='set_cpus', job=job_name, before=old_cpus, after=ncpu)
   # log('run.jsonl', 'execution', action='job_end', job=job_name, status=str(job.status))
   # log('run.jsonl', 'verification', item='rows', expected='>0', actual=n_rows, result='ok')
   ```

4. **スクリプトの版**は、スクリプトを git で管理していればコミットの番号を残すのが一番確実。管理していなければ、スクリプト自体のハッシュ値を残す。
5. **最初は一覧と詳細を作り込まない。** JSON Lines を残すことだけ先に始めれば、一覧・詳細レポート（HTML）は後からそのログを読んで作れる。

### 間違っているとしたら、ここ

- ログの項目を最初から全部そろえようとすると、自動化そのものが進まない。最初は「入力ファイルのハッシュ値・設定の前後・終了状態・抽出の条件」の4つから。
- チェック担当者が実際に何を見て合否を決めているかを聞かずに検証ログの項目を決めると、使われないログになる。項目は、チェック担当者に1回見てもらって決める。

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

- [abapy tutorial（abaqus python はトークンを使わない、という説明）](https://abapy.readthedocs.io/en/latest/tutorial.html)
- [ABAQUS python tutorial（Brown University）](https://www.brown.edu/Departments/Engineering/Courses/En1750/Tutorials/ABAQUS_python_tutorial.pdf)
- [Understanding the files generated by creating and analyzing a model（.lck の役割。Abaqus 2017 文書）](https://abaqus-docs.mit.edu/2017/English/SIMACAECAERefMap/simacae-c-dbsconcfilesgenerated.htm)
- [Opening a model database or an output database（計算中の ODB を開くときの注意）](https://abaqus-docs.mit.edu/2017/English/SIMACAECAERefMap/simacae-t-dbsmdbopen.htm)
- [OdbError と .lck（Eng-Tips）](https://www.eng-tips.com/viewthread.cfm?qid=477078)

- [Abaqus/CAE execution（recover・script・noGUI などの起動オプション。Abaqus 2016 文書）](https://ceae-server.colorado.edu/v2016/books/usb/pt01ch03s02abx07.html)
- [Recreating an unsaved model database（.jnl の役割。Abaqus 2017 文書）](https://abaqus-docs.mit.edu/2017/English/SIMACAECAERefMap/simacae-c-dbsfilesrecover.htm)
- [Abaqus/CAE command summary（-- の後の引数を sys.argv で受け取る例）](https://abaqus-docs.mit.edu/2017/English/SIMACAEEXCRefMap/simaexc-c-caeproc.htm)
- [Job object（numCpus・numDomains・waitForCompletion。Abaqus 2025 文書）](https://docs.software.vt.edu/abaqusv2025/English/SIMACAEKERRefMap/simaker-c-jobpyc.htm)
- [複数ジョブを並行して流す書き方（Abaqus 2016 文書）](https://ceae-server.colorado.edu/v2016/books/cmd/pt02ch06s08.html)
- [.cae が壊れたときの作り直し（caeassistant、第三者）](https://caeassistant.com/questions/tags/load/)

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
