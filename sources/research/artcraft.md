# ArtCraft（Crafting Apps）: Adobe などの代替を名乗る Rust 製オープンソースのアプリ群（2026-10 調査）

**grounding**: paraphrase・要検証。ユーザーから共有された文章（動画で紹介されたとのこと。元の動画は未確認）の内容を、
GitHub の組織ページ（storytold）と各リポジトリのページを WebFetch で取得して確かめた。取得したのはページの要約で、
README・LICENSE・ソースの本文は読んでいない。getartcraft.com はネットワークの制限で開けなかった。
Web 検索の要約だけでは一部のアプリが見つからなかったので、実在の確認は GitHub のページで行った。

## 結論

「ArtCraft（Crafting Apps）」は、GitHub の組織 storytold が出している、Adobe などの製品の代替を名乗る Rust 製オープンソースのアプリ群。
共有された7つのアプリと ArtCraft 本体は、いずれも storytold に実在する（2026-10-07 時点）。

| アプリ | 代替する製品 | リポジトリ（github.com/storytold/） | stars（概数） |
|---|---|---|---|
| PhotoCraft | Photoshop | photocraft | 11.9k |
| FilmCraft | Premiere Pro | filmcraft | 2.3k |
| LightCraft | Lightroom | lightcraft | 1.9k |
| PrintCraft | Acrobat | printcraft | 1.7k |
| VectorCraft | Illustrator | vectorcraft | 1.5k |
| EffectCraft | After Effects | effectcraft | 1.2k |
| DesignCraft | InDesign | designcraft | 711 |
| ArtCraft（本体） | — | artcraft | — |

- EffectCraft はリポジトリに説明文がなく、After Effects の代替かどうかは共有された情報のとおり（ページでは確認できていない）。
- 各リポジトリは「clean-room reimplementation（クリーンルームでの再実装）… in pure Rust」と説明されている。
- 組織にはほかに CADCraft（AutoCAD 風）、WordCraft、GridCraft（Excel 風）、SoundCraft（Pro Tools）、DeckCraft（PowerPoint）がある。
  「Fonts for the Crafting Apps」（craft-fonts）もあり、「Crafting Apps」の呼び名はここで裏付けが取れた。組織のリポジトリは全部で41。
- ArtCraft 本体の説明は「artists, designers, and filmmakers 向けの intentional crafting engine」。検索結果の要約では、AI で映像を作るツール
  （仮想の撮影セット、2D キャンバスの inpaint/outpaint、画像・動画の生成モデルをまとめて使う機能）として紹介されている。

## 未確認

- **ライセンス**: 組織の一覧は Apache-2.0、printcraft・designcraft のページは「MIT または Apache-2.0 の二重」と表記が違う。各リポジトリの LICENSE で確かめる。
- **成熟度**: 第三者の投稿が「vibe-coded（AI に書かせた）」と評している（検索結果の要約。元の投稿は未読）。実用に耐えるかは未確認。
- ビルド手順・対応 OS（README で確認）、getartcraft.com の中身、紹介動画の URL。
