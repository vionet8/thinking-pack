# thinking-pack

私の考え方と集めた知見を、どのClaudeでも同じ前提で使うためのプラグイン。
中身（知識パック）と、それを扱う道具（スキル・起動時の読み込み）を一緒に運ぶ。

## 入っているもの

| 部品 | 役割 |
|---|---|
| スキル `thinking-core` | 考え方の核とClaudeへの頼み方。相談・調べ物・読み解きのときに使われる。資料（知識パック6ファイル）を同梱 |
| スキル `record-thought` | 本人の考えを決まった形で記録する。職場では保存せず、家に持ち帰るメモとして出す |
| スキル `source-summary` | 動画・記事の要約を、出典の段階つき・丸写しなしで作る |
| 起動時の読み込み（hooks） | Claude Codeでは、セッション開始時に核（`00_CORE.md`）を自動で読み込む |

入っていないもの: 家族・お金・仕事の中身、noteのペンネームとアカウント、個人のアカウントや
APIキーが要る道具（Gmail・ドライブ・Geminiでの動画取得・AI秘書）。これらは家の環境だけで使う。

## 職場での入れ方

### Claude Code

```
/plugin marketplace add vionet8/thinking-pack
/plugin install thinking-pack@thinking-pack
```

そのあと `/plugin` の **Marketplaces** で thinking-pack を選び、**Enable auto-update** を
オンにする。家で蓄積が増えるたびに、職場側も自動で最新になる。

### claude.ai（ブラウザ・アプリ）

会社のプランで個人のスキルやプラグインの追加が許可されていれば、同じものを入れられる
（claude.ai では読み込まれる部品が一部違い、起動時の自動読み込みは使えない）。
許可されていなければ、プロジェクトのナレッジに `skills/thinking-core/references/00_CORE.md` を
アップロードする。

## 守ること

- 情報は「家 → 職場」の一方向。職場で出た考えは `record-thought` の持ち帰りメモにして、
  会社の情報を抜いてから家のClaudeに貼る。
- このリポジトリは公開なので、ここに入るのは知識パックとスキルだけにする。
