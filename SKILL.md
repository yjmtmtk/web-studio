---
name: web-studio
description: >-
  ゼロからWebサイト/ページ(ランディング、コーポレート/ブランド、LP、ポートフォリオ、複数ページ)を
  デザイン品質込みで制作する依頼で発火。「サイト作って」「LP作って」「ホームページ作って」「ポートフォリオ
  作って」など、ゼロからのWeb制作・デザイン品質が問われる静的サイトで使う。型付き Astro + Tailwind と
  視覚自己批評ループで作る。既存サイトの小改修や、SSR/動的アプリが主体のケースには別手段を。
---

# Web Studio

ゼロからのWeb制作スタジオ。設計思想は2つの賭けに集約される:

1. **AIが最も得意な土俵を使う。** bespoke DSL でなく **Astro + Tailwind + 型付き .astro コンポーネント**。
   モデルが訓練データで最も見ていて(=分布内で精度↑)、型が契約を**強制**し、ツール(型チェック/lint/
   画像最適化/a11y/perf)が桁違いに厚い。Tailwind は**実ビルドで本物のCSSに**(ランタイムJIT/FOUCなし)。
2. **自分の出力を“見て”直す。** 一発生成で終えず、**視覚自己批評ループ**で人間の2巡目を再現する。これが
   品質の最大レバー。

構成要素:**Astro/Tailwind**(土台)・**型付きコンポーネント・キット**(あなたのデザインDNA)・
**unsplash-fetch**(実写真)・**あなたの判断**。

このドキュメントが手法そのもの。フェーズ分割・PLAN承認・出力ファイル構成の事前提示・デザイン規律・
**自己批評ループ・ゲート段**は、品質と速度を両立させる技術。守ること。

## ⚠ サイレントな罠(全フェーズ共通・静止スクショでは気づけない)

このスキルの失敗はほぼこの4つに集約される。自己批評ループが**空セクションを撮って「合格」**を出す主因でもあるので、着手前に頭へ入れる(詳細は各フェーズ):
- **撮影前に reveal を可視化しないと空撮り**になる → 撮影モード(フェーズ4)。最優先。
- **Tailwind v4 JIT が JS連結の class を拾わず CSS が出ない** → 意味クラス1つ+CSS変数(規約)。
- **使う書体を `<head>` で読まないと見出しが無言フォールバック** → Layout で実読込(フェーズ2)。
- **npm クールダウンで create-astro / `astro add` / `npx` ツールが失敗** → ピン下げ + 手動配線 + `min-release-age=0` 上書き(セットアップ)。

## セットアップ(1プロジェクト1回・**バックグラウンド先行**)

**まず1回 `npm config get min-release-age before` を確認**——どちらか設定済みなら**クールダウン環境**。下記①②③の回避を**最初から織り込んで**から動く(暗号的な install 失敗を待たず・BG レーンを沈黙させない)。
PLAN 着手と同時に**BG 起動**し、承認待ち＝死に時間に install を隠す(scaffold はテーマ非依存=絶対に無駄に
ならない)。リッチ既定ライブラリと型ゲートも**この1回でまとめて先入れ**し、後段の install 停止をゼロにする:

```bash
npm create astro@latest <proj> -- --template minimal --install --typescript strict --yes
cd <proj> && npx astro add tailwind --yes   # v4 では @tailwindcss/vite を astro.config に自動追加(別途 init 不要)
npm i gsap && npm i -D @astrojs/check   # モーション既定+型ゲートを先行(カルーセルは native scroll-snap で自作・lib不要)
# 開発: npx astro dev   ビルド: npx astro build → dist/   検証: npx astro check
```
- **Tailwind の実体は `@tailwindcss/vite`**(v4)。`astro add tailwind` がこれを `astro.config` に挿す=`tailwind.config` も `@tailwind` ディレクティブも不要。CSS 側は `@import "tailwindcss"` 1行だけ(後述 `global.css`)。
- **npm クールダウン環境**(`min-release-age`/`before`)は3箇所を壊す:①create-astro が ETARGET → `package.json` の `"astro"` を **`^X.Y.0` に下げて** install し直す。②**`astro add <x>` も依存解決に失敗**(config 未配線・CSS 未生成のまま落ちる)→ 統合を手動 install して `astro.config` に挿す(例:`@tailwindcss/vite` を入れ `vite:{plugins:[tailwindcss()]}`)+ `global.css` 自作。③**`npx <tool>`(unsplash-fetch 等)が "No versions available" でブロック**(キーがあっても)→ その呼び出しだけ **`npm_config_min_release_age=0 npx …`** で上書き(グローバル設定は不変)。
- 要件:Node。画像は **`npx unsplash-fetch`**(導入不要・`UNSPLASH_ACCESS_KEY` 設定)。

## 能動的アーキテクト + フェーズ(並列レーンで走らせる)

重い待ち(**npm install・画像API**)は創造的作業の裏に隠す。クリティカルパスは *設計→キット→ページ→批評* だけ。

| レーン | 役割 | 出力 | 並列性 |
|---|---|---|---|
| **0 足場(BG)** | 環境 | scaffold + 全lib install | PLAN着手と**同時にBG起動**、承認待ちに隠す |
| **1 設計** | アーキテクト | `PLAN.md` + **画像マニフェスト** | 仕様を書く・**承認で止まる** |
| **B 画像(BG)** | 取得 | `public/img/*` 実画像 | マニフェスト確定後、**別レーンで並行取得** |
| **2 トークン+キット** | 基盤 | `@theme` + 型付き `.astro` | **直列**=唯一のスパイン・最小で即凍結 |
| **3 ページ合成** | 実装 | `src/pages/*.astro` | **並列**(1ページ1エージェント・プレースホルダ上で) |
| **4 自己批評** | 批評+修正 | 改善された各ページ | **並列**(ページ毎に撮影→批評) |
| **5 ビルド+ゲート** | 検査 | `dist/` + 緑のゲート | — |

**キットだけが直列スパイン**:最小契約で速く凍結したら、ページは即ファンアウト。
**画像はパスが契約**:ビルダーはプレースホルダで即ビルドし、画像レーンが同じパスへ実画像を流し込む(無編集スワップ)。
コンポーネントが `aspect-[…]` で比率を固定するので**後差しでもレイアウトシフトしない**。

---

# フェーズ1 — 設計 → `PLAN.md`(承認まで止まる)

*要約:依頼を `PLAN.md`(ページ構成+セクション+画像マニフェスト)に落とし、承認を取る。並行で足場と画像のBGレーンを起動。*

**目標**:依頼を完全な実装設計図に変換する。入力はユーザー指示(無ければ `ORDER.md` を参照)。出力は
**プロジェクトルートの `PLAN.md`**(`src/` の中には置かない)。

**3パスで思考する**:(1) 目的・ターゲット・必要機能を理解 →(2) ページ群と共通/再利用パーツを洗い出す →
(3) 各ページのセクション・内容・要件を詳細化。

## `PLAN.md` の書式(制作指示書)

```markdown
# サイトの基本情報
## テーマとターゲットユーザー        [具体的なテーマと、誰のためか]

# デザインテイスト
## 全体的なデザインスタイル          [例:モダンミニマル、エディトリアル]
## 色彩計画                          [役割: bg / ink / brand / accent / surface]
## タイポグラフィ & インタラクション [書体、操作/アニメーションの意図]

# ヘッダー（全ページ共通）           [ロゴ、ナビ項目、CTA]
# フッター（全ページ共通）           [著作権、連絡先、SNS、クレジット]

# ページ構成
- トップ        src/pages/index.astro
- [その他]      src/pages/[name].astro

# 各ページのセクション内容

## [ページ名] (src/pages/[name].astro)

### ページ図   （全ページ必須・アスキーアートのワイヤーフレーム）

### [セクション名]
#### コンポーネント: <Hero> / <Features> / <Gallery> 等（キットから選ぶ。無ければ新コンポーネント名）
#### 要件: 渡す Props（例 heading, items[]）と内容・挙動
```

**命名規則**:ページ=`src/pages/<name>.astro`、共通=`Layout`/`Header`/`Footer`。セクションは**再利用可能な
キットコンポーネント**(`Hero`/`Features`/`Gallery`/`CTA`…)を**選んで Props で構成**する(ページ毎に新ファイルを
作らず合成)。本当に固有のものだけ新コンポーネントを足す。

最後に**出力ファイル構成**を**先に提示**(`//` 縦揃え、コンポーネント漏れ注意):

#### 出力ファイル構成
```text
PLAN.md                          // 設計図（ルート・非出力）
src/styles/global.css            // @theme トークン + base
src/layouts/Layout.astro         // <head>(OGP契約)+ Header/<slot>/Footer
src/components/Header.astro       // ナビ（Astro.url で自動アクティブ）
src/components/Footer.astro
src/components/Hero.astro          // 型付きキット
src/components/Features.astro
src/components/Gallery.astro
src/components/CTA.astro
src/pages/index.astro              // データ駆動で合成
public/img/                        // unsplash-fetch（root絶対 /img/...）
public/favicon.svg                 // 必ず生成
```
## 画像マニフェスト(ビルダー ⇄ 画像レーンの契約)

PLAN に**各画像スロット**を列挙:`パス | 被写体(英語keyword) | 比率/向き | alt`。これが契約なので、ビルダーは
プレースホルダで即ビルドでき、画像レーンが同じパスへ実画像を流し込む:

#### 画像マニフェスト
```text
/img/hero.jpg    | seascape dusk | 16/9 横 | 夕暮れの海
/img/room-1.jpg  | ryokan room   | 3/4 縦  | 客室
…
```

**並列レーンを起動**:PLAN 着手と同時に**レーン0(scaffold+install)を BG 起動**。マニフェスト確定後すぐ
**レーンB(画像取得)を BG 起動**(各スロットへ `npx unsplash-fetch --keyword … --output public/img --name <slot>`)。承認を待つ間に両レーンが終わる。

**ここで止まる。承認を得る。**

---

# フェーズ2 — トークン + 型付きキット(直列・契約を速く凍結)

*要約:`@theme` トークンと型付き `.astro` コンポーネントを最小契約で固める。ここだけ直列、固めたら即ファンアウト。*

唯一の直列依存は**デザインシステム**。これを1手で強く固めれば、以降は並列にできる。**最小契約で凍結**(完璧を
求めない)し、すぐページへファンアウト。画像は**プレースホルダ**(`bg-surface` のボックス)を参照し、画像レーンを待たない。

**`src/styles/global.css`** — Tailwind v4 の `@theme` でトークンを定義(=Tailwindユーティリティ化):
```css
@import "tailwindcss";
@theme {
  --color-bg: #f7f6f3; --color-ink: #1b1b18; --color-muted: #6b6b63;
  --color-brand: #4a5d4f; --color-surface: #eceae4; --color-line: #ddd9d0;
  --font-display: "Fraunces", Georgia, serif; --font-body: system-ui, -apple-system, sans-serif;
}
body { background: var(--color-bg); color: var(--color-ink); font-family: var(--font-body); }
h1,h2,h3 { font-family: var(--font-display); font-weight: 400; letter-spacing: -.01em; }
```
→ `bg-bg` `text-ink` `text-muted` `text-brand` `bg-surface` `border-line` `font-display` が**型のように効く契約**になる。

**コンポーネント・キット**(`src/components/*.astro`)— 各ブロックは **TypeScript Props interface=強制された契約**。
ゼロから書かず「選ぶ→テーマ→中身を埋める」:
```astro
---
interface Props { overline?: string; heading: string; subheading: string; ctaLabel: string; ctaHref: string; image: string; }
const { overline, heading, subheading, ctaLabel, ctaHref, image } = Astro.props;
---
<section class="relative flex min-h-[82vh] items-center justify-center text-center text-white"
  style={`background-image:linear-gradient(rgba(20,20,18,.30),rgba(20,20,18,.68)),url(${image});background-size:cover;background-position:center;`}>
  <div class="mx-auto max-w-3xl px-6">
    {overline && <p class="mb-5 text-xs uppercase tracking-[0.25em] text-white/70">{overline}</p>}
    <h1 class="font-display text-5xl leading-[1.05] sm:text-7xl">{heading}</h1>
    <p class="mx-auto mt-6 max-w-xl text-lg text-white/90 sm:text-xl">{subheading}</p>
    <a href={ctaHref} class="mt-10 inline-block rounded-sm bg-brand px-8 py-3.5 text-xs uppercase tracking-widest text-white">{ctaLabel}</a>
  </div>
</section>
```
- **Layout.astro** は `<head>`(OGP完全契約 + fonts + favicon)+ Header + `<slot/>` + Footer。
  **`--font-display` 等で使う書体は `<head>` で実際に読む**(例:`<link rel="preconnect" href="https://fonts.googleapis.com"><link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600&display=swap" rel="stylesheet">`)。読み込み漏れは見出しが Georgia へ**無言フォールバック**する沈黙トラップ。
- **Header.astro** は `Astro.url.pathname` で**アクティブナビを自動**(`aria-current="page"`)。**モバイル(`<768`)はナビのトグルを必ず置く**——デフォルトはゼロJSの最小形、凝った演出は指示時のみ(レシピ:レスポンシブナビ)。
- **凍結したら『キット契約』を出す**(= web-forge の `style-guide.yaml` 役):各コンポーネント → **Props 署名1行** + トークン名 + 画像スロットを `PLAN.md` へ `## キット契約` として追記。**唯一の真実**は `global.css`(トークン)と `src/components/*.astro`(Props)で、契約はその索引。フェーズ3の各エージェントは着手前に必ず読む。

---

# フェーズ3 — ページ合成(並列・データ駆動)

*要約:キットにデータを渡して各ページを合成。1ページ1エージェントで並列、共有ファイルは触らない。*

`src/pages/*.astro` でキットを**データ駆動**に合成。配列を型付き Props に渡し、コンポーネント側で `.map`:
```astro
<Features overline="Approach" heading="How we work" lead="…"
  items={[ { title:'Light first', body:'…' }, { title:'Honest materials', body:'…' } ]} />
<Gallery overline="Selected work" heading="Recent projects"
  projects={[ { image:'/img/work-1.jpg', title:'Fjord House', meta:'Bergen · 2024' } ]} />
```
ページは**合成のみ**。各ページは独立 → **1ページ1エージェントでファンアウト**(並列)。実画像がまだでも
**プレースホルダのまま合成・ビルドして可**(画像レーンが後差し)。

**各エージェントへの手渡し(コア把握→役割判断)**:各ページエージェントは契約を**継承しない別コンテキスト**。ファンアウト時に **(a) `## キット契約`(Props署名・トークン・画像スロット)、(b) 担当ページの PLAN セクション** を渡し、**「`global.css` と使うコンポーネントを読んで把握してから合成」**を課す(不明点は実ファイルが真実)。渡さないと推測で型不一致→`astro check` で手戻り。担当は自分のページ1枚(下記不変条件)。

**並列の不変条件(衝突防止)**:ファンアウトは**キット凍結後にのみ起動**する。各エージェントは**自分の `src/pages/<name>.astro` だけを書く**。共有ファイル(`global.css`・`Layout`・共通コンポーネント)は**読み取り専用**——不足や修正が要るキットは並列を止め、直列スパインに戻して直してから再ファンアウトする(複数エージェントの同時追記は競合する)。

---

# フェーズ4 — 視覚自己批評ループ(本スキルの目玉)

*要約:ビルド→2幅でスクショ→マルチモーダルで批評→ピンポイント修正→再ビルド。停止条件を全 Yes で満たすまで。*

一発生成で終えない。**自分の出力を見て直す**:

1. `npx astro build` → `dist/` を静的サーバで配信(`python3 -m http.server -d dist`)。
2. 各ページを **Playwright で全画面スクショ**(**撮影モード**で reveal の隠し要素を可視化してから撮る=下のレシピ)。
   **デスクトップ(1440)とモバイル(375)の両幅で撮る**(`browser_resize` で切替)。片幅だけ整えるとモバイル崩れを見逃す。
   ページ数が多ければ**ページ毎に並列**(各エージェントが配信→撮影→批評)。
3. **そのスクショを自分(マルチモーダル)で読み**、ルーブリックで批評 → **具体的な修正リスト**を出す
   (例:ヒーローのリード可読性↓→オーバーレイ下重め+リード拡大、暗い背景上のヘッダーが低コントラスト→
   配色を適応化、ギャラリーが孤立→列数を整える)。
4. 修正を**ピンポイントで**当てる(並列可)→ 再ビルド → 再スクショ。
5. **停止条件は二値で判定**(下記)。満たすまで、または予算切れまでループ。重要導線(ヒーロー/主要CTA)は厚く、定型は薄く。

**停止条件(各幅でこの問いを Yes/No で答え、全 Yes なら合格)**:
- [ ] 階層:最重要要素が**サイズと位置だけ**で一目で立っているか(太字頼みでないか)
- [ ] 余白:垂直リズムが一定軸に吸着し、孤立/詰まりがないか
- [ ] 可読性:全テキストが背景に対し十分なコントラスト(本文 4.5:1 目安)か
- [ ] レスポンシブ:375 幅で横スクロール・はみ出し・重なりが無いか
- [ ] 一貫性:同種要素(カード/ボタン/見出し)の様式が揃っているか
- [ ] 品位:モーション/装飾が過剰でなく、ブリーフのトーンに合うか

**巡回回数の目安**:重要導線ページは最低2巡(初回批評→修正→再批評で確認)、定型ページは1巡で可。全 Yes に達したら**それ以上回さない**(過剰研磨で予算を溶かさない)。

---

# フェーズ5 — ビルド + ゲート段(早い順に落とす)

*要約:型→ビルド→(任意で a11y/perf/視覚)の順にゲートを通す。緑になった `dist/` が成果物。*

`npx astro check`(型=契約)→ `npx astro build`(=lint/コンパイル/画像最適化込み)→ 任意で `axe`(a11y)・
Lighthouse(perf 予算)・視覚ルーブリック。`dist/` が成果物(純静的・CDN可)。

---

# 画像・デザイン・規約

- **画像 = `npx unsplash-fetch`(導入不要・要 `UNSPLASH_ACCESS_KEY`)**。`public/img/` に取得し **root絶対
  `/img/...`** 参照、`alt` 必須、出力 JSON の `attribution.markdown` をフッターのクレジットへ。favicon は
  `public/favicon.svg` を**必ず生成**(最小例:`<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32"><rect width="32" height="32" rx="6" fill="#4a5d4f"/><text x="16" y="23" text-anchor="middle" font-size="18" fill="#fff" font-family="Georgia,serif">N</text></svg>` の要領でブランド頭文字+brand色)。
  - 取得:`npx unsplash-fetch --keyword "<英語keyword>" --output public/img --name <slot> [--width 1600]`
  - **テーマ厳選(盲目的 index 0 にしない)**:`--map-only` で30枚のHTMLコンタクトシート
    (`_unsplash-cache/<kw>-map.html`)を出し→**配信→スクショ→Read**で最良を選び→`--index N` で取得
    (`file://` 不可なら `npx -y serve _unsplash-cache`)。被写体がブリーフに合うか**目視確認**(index 0 任せは的外れを掴む)。
  - キャッシュ `_unsplash-cache/` は **`.gitignore`** に。
- **ビジュアル = White Space 6原則(指示がない限り採用)**。プロのレイアウトは「足す」より「制限する」:
  1. **階層**=太字を封印し、**サイズと位置のみ**で作る(装飾でなく構造の美)。
  2. **垂直軸**=起点を極少数の軸に限定し**吸着**(迷いのない視線誘導・強い清潔感)。
  3. **モジュラースケール**=数値を**特定比率(黄金比等)のみ**に縛る(数学的な調和とリズム)。
  4. **ネガティブシェイプ**=余白を先に**「主役の図形」として配置**(大胆で贅沢な空間)。
  5. **ベースライングリッド**=全行を**一定の垂直リズムに完全吸着**(建築的秩序・信頼感)。
  6. **ボーダレス近接**=枠線を廃し、要素間の**「距離」のみで分類**(ノイズ極小のモダンな透明感)。
  高級感は半透明・backdrop・微グラデで補強。**この静的レイアウトの規律と上の動的リッチネスは両立する**
  (端正な構図 × 上品なモーション)。
- **リッチネス = 既定**(指示がない限り積極的に)。スクロール連動・パララックス・リビール・カルーセル・
  ホバーを**標準で盛り込む**。自前実装せずライブラリで:
  - **モーション**:**GSAP + ScrollTrigger** を **native スクロール上**で(happy path)。軽量なら **AOS**。
  - **カルーセル**:既定は **native scroll-snap**(`flex snap-x snap-mandatory overflow-x-auto` + `scrollBy()`)。凝るなら **Swiper/Embla/Blossom**。
  - **慣性スクロールは既定にしない**(native が a11y/sticky/アンカーと噛む)。要れば **Lenis** を別途、滑らかさだけなら `scroll-behavior:smooth`。
  - **React エフェクト**:凝った演出(**React Bits** 等)は **`@astrojs/react` のアイランド**で、ブリーフに合う時だけ。
  - **節度**:`prefers-reduced-motion` 尊重、遅延読み込みで LCP を阻害しない。リッチ≠うるさい。
- **スタイル = Tailwind が土台(必須)**。`@tailwindcss/vite` で**実ビルドし本物の
  CSS にコンパイル**(ランタイムJIT/FOUC なし・純静的)。トークンは `@theme` で定義=**契約層**。スタイルは
  Tailwind ユーティリティを主とし、込み入った所だけ `.astro` の scoped `<style>` を併用。
- **規約**:見出し=`font-display`・本文=`font-body`、トークンのみ(色/余白/書体の直書き禁止)、padding で余白、セマンティックHTML、
  クライアント操作は **Astro islands**(Alpine/React/Svelte を必要な所だけ)。

---

# 統合レシピ(調査せず貼って使う)

ニッチ部分の調査時間をゼロにする検証済みスニペット。

## プレースホルダ画像(実画像を待たない)
画像レーン完了前は各スロットを**単色ボックス**で代替してレイアウトを確定 → 実画像が来たら `<img>` に差し替え:
```astro
<div class="aspect-[3/4] w-full bg-surface"></div>
```
コンポーネント側を `aspect-[…]` 固定＋`object-cover` にしておけば、実画像差し込みで**シフトしない**。

## 撮影モード(full-page スクショ)
**自己批評ループの前提**:scroll-reveal の隠し要素を可視化せずに撮ると、ループは**空セクションを撮って誤批評**する(沈黙する罠)。撮影直前に可視化(Playwright `browser_evaluate`)。**インライン `style.opacity='1'` では不十分**——GSAP の rAF が `fullPage` 撮影中のビューポート再設定で reveal を再 tween し、インライン値を上書きして淡く写る。**`<style>` に `!important` ルールを注入**して殴る(GSAP のインライン非 important opacity に確実に勝つ)。セレクタは reveal 規約に合わせる(GSAP `[data-reveal]`、AOS `[data-aos]`、`.opacity-0` 等):
```js
async () => {
  const s = document.createElement('style');
  s.textContent = '[data-reveal],[data-aos],.opacity-0{opacity:1!important;transform:none!important;visibility:visible!important;}';
  document.head.appendChild(s);
  if (document.fonts?.ready) await document.fonts.ready; await new Promise(r=>setTimeout(r,500));
}
```

## レスポンシブナビ(モバイルメニュー=定石)
**指示がなくてもモバイルメニューは必ず付ける**(デフォルト=下記・ゼロJSの最小形)。フルスクリーン上書き等の演出は**ブリーフが求める時だけ**拡張(リッチ≠うるさい)。要点:**`<nav>` は1つだけ=同じリンクをDOMに重複させない(SEO)**——デスクトップは横並び、モバイルは `<768` でドロップダウンへ CSS リフロー/active は `aria-current="page"`+`aria-[current=page]:`/ハンバーガーは `currentColor`=`--hdr-fg` 継承でヒーロー上でも視認。
```astro
---
const path = Astro.url.pathname;
const nav = [{label:'Work',href:'/work'},{label:'Studio',href:'/studio'},{label:'Contact',href:'/contact'}];
const active = (h: string) => (h === '/' ? path === '/' : path.startsWith(h));
---
<header data-header class="fixed inset-x-0 top-0 z-50">
  <div class="mx-auto flex max-w-6xl items-center justify-between px-6 py-5">
    <a href="/" class="font-display text-xl">BRAND</a>
    {/* ゼロJS peer checkbox。DOM順は checkbox → label → nav(peer は対象より前) */}
    <input id="nav-t" type="checkbox" class="peer sr-only" aria-label="メニュー" />
    <label for="nav-t" class="cursor-pointer space-y-1.5 peer-focus-visible:outline-2 peer-focus-visible:outline-brand md:hidden" aria-hidden="true">
      <span class="block h-0.5 w-7 bg-current"></span><span class="block h-0.5 w-7 bg-current"></span>
    </label>
    {/* リンクは1セット。mobile=ドロップダウン / md=ヘッダー内の横並びに復帰 */}
    <nav aria-label="Primary" class="absolute inset-x-0 top-full hidden flex-col gap-1 bg-bg p-6 shadow-lg peer-checked:flex md:static md:inset-x-auto md:top-auto md:flex md:flex-row md:items-center md:gap-8 md:bg-transparent md:p-0 md:shadow-none">
      {nav.map((n) => <a href={n.href} aria-current={active(n.href) ? 'page' : undefined} class="py-2 text-lg md:py-0 md:text-sm aria-[current=page]:text-brand">{n.label}</a>)}
    </nav>
  </div>
</header>
```
**拡張(指示時のみ)**:フルスクリーン上書き(`fixed inset-0`)・☰→✕ 変形・`body` スクロールロック・Esc/リンクで閉じる→小さな島(`<script>`)で足す。`prefers-reduced-motion` 尊重。`ClientRouter` 不使用=遷移ごとの再初期化は不要(MPA リロードで状態リセット)。

## Tailwind v4 の確実な書き方
- 比率/拡大は**角括弧**で確実に:`aspect-[3/4]` `aspect-[4/5]` `scale-[1.15]`。トークン色は `@theme` から `bg-bg` `text-ink` 等が自動生成。
- **ランタイムで class をトグルする箇所**(ヘッダーのスクロール状態・配色適応など)は **Tailwind ユーティリティを
  JS 文字列で足さない**(スキャン漏れで CSS が出ない)。`is-scrolled`/`over-hero` 等の**意味クラスを1つトグル**し、
  色は **CSS変数**で global.css 側に当てる(例:`[data-header].over-hero:not(.is-scrolled){ --hdr-fg:#fff }`)。
