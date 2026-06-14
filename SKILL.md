---
name: web-studio
description: >-
  ゼロからWebサイト/ページを制作するワークフロー(ランディング、コーポレート/ブランド、LP、
  ポートフォリオ、複数ページ)。「サイト作って」「LP作って」「ホームページ作って」など、デザイン品質が
  問われるゼロからのWeb制作で発火。**AIが最も得意な土俵**で組む:Astro + Tailwind + 型付き .astro
  コンポーネント(Props=契約)+ デザイントークン。画像は unsplash-fetch。最大の特徴は **視覚自己批評
  ループ**(ビルド→スクショ→マルチモーダル批評→ピンポイント修正→再ビルド)で品質を閉ループ改善する点。
  静的サイトに最適。
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

## セットアップ(1プロジェクト1回・**バックグラウンド先行**)

PLAN 着手と同時に**BG 起動**し、承認待ち＝死に時間に install を隠す(scaffold はテーマ非依存=絶対に無駄に
ならない)。リッチ既定ライブラリと型ゲートも**この1回でまとめて先入れ**し、後段の install 停止をゼロにする:

```bash
npm create astro@latest <proj> -- --template minimal --install --typescript strict --yes
cd <proj> && npx astro add tailwind --yes   # v4 では @tailwindcss/vite を astro.config に自動追加(別途 init 不要)
npm i gsap lenis swiper && npm i -D @astrojs/check   # モーション既定+型ゲートを先行(carousel は PLAN に合わせ swiper/embla/blossom)
# 開発: npx astro dev   ビルド: npx astro build → dist/   検証: npx astro check
```
- **Tailwind の実体は `@tailwindcss/vite`**(v4)。`astro add tailwind` がこれを `astro.config` に挿す=`tailwind.config` も `@tailwind` ディレクティブも不要。CSS 側は `@import "tailwindcss"` 1行だけ(後述 `global.css`)。
- **npm クールダウン環境**(`min-release-age`/`before`)では create-astro の最新ピンが弾かれる(ETARGET)。
  `package.json` の `"astro": "^X.Y.Z"` を **`^X.Y.0` に下げて** install し直す。
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

最後に**出力ファイル構成**を `text:出力ファイル構成` で**先に提示**(`//` 縦揃え、コンポーネント漏れ注意):

```text:出力ファイル構成
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
```text:画像マニフェスト
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
  --font-display: "Fraunces", Georgia, serif;
}
body { background: var(--color-bg); color: var(--color-ink); font-family: system-ui, sans-serif; }
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
- **Header.astro** は `Astro.url.pathname` で**アクティブナビを自動**(中継不要)。

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
  `public/favicon.svg` を**必ず生成**。
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
- **リッチネス = 既定(指示がない限り積極的に)**。スクロール連動アニメ・パララックス・要素のリビール・
  カルーセル/スライダー・スムーススクロール・ページ遷移・ホバーのマイクロインタラクションを**標準で盛り込む**。
  自前実装せず**ライブラリをふんだんに使う**:
  - **モーション/スクロール**:**GSAP + ScrollTrigger**(連動・pin・パララックス)、**Lenis**(スムーススクロール)、
    **AOS**(軽量リビール)。
  - **カルーセル**:**Swiper** / **Embla**、または **Blossom Carousel**(ネイティブ scroll-snap ベース・~4kb・
    タッチで0kb・scroll-snap/sticky/スクロール駆動アニメと併用可)。
  - **ページ遷移**:Astro **`<ClientRouter />`**(`astro:transitions`・SPA風)。
  - **読み込み**:バニラ系(GSAP/Swiper/Lenis/AOS/Blossom)はコンポーネント `.astro` 内の `<script>`(Astro が
    バンドルしクライアント実行)。**React 製は `@astrojs/react` を入れアイランド**(`client:visible`/`client:idle`)で。
    `<ClientRouter />` 使用時はスクロール系を **`astro:page-load`** で再初期化。
  - **知っておく(常用しない)**:**React Bits**(reactbits.dev)= Aurora/Particles/Split-Text 等の**独特な**アニメ
    React コンポーネント集。見た目が強く出る(使いすぎると没個性化)ので**ブリーフに合う時だけ選択的に**
    (使うなら `@astrojs/react` のアイランドで)。
  - **節度**:`prefers-reduced-motion` を尊重、遅延読み込みで **LCP を阻害しない**。リッチ≠うるさい——
    上品さは保つ。明示的に抑制を指示された時のみ簡素化する。
- **スタイル = Tailwind が土台(必須)**。`@tailwindcss/vite` で**実ビルドし本物の
  CSS にコンパイル**(ランタイムJIT/FOUC なし・純静的)。トークンは `@theme` で定義=**契約層**。スタイルは
  Tailwind ユーティリティを主とし、込み入った所だけ `.astro` の scoped `<style>` を併用。
- **規約**:見出し=`font-display`、トークンのみ(色/余白の直書き禁止)、padding で余白、セマンティックHTML、
  クライアント操作は **Astro islands**(Alpine/React/Svelte を必要な所だけ)。

---

# 統合レシピ(調査せず貼って使う)

ニッチ部分の調査時間をゼロにする検証済みスニペット。

## モーション基盤 `src/lib/motion.ts`(Lenis + GSAP ScrollTrigger)
Layout の `<script>import '../lib/motion.ts'</script>` で読込み。`astro:page-load`/`before-swap` で View Transitions に追従。
```ts
import Lenis from 'lenis';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
let lenis: Lenis | null = null;
const raf = (t: number) => lenis?.raf(t * 1000);
function setup() {
  if (!reduce) { lenis = new Lenis({ lerp: 0.1 }); lenis.on('scroll', ScrollTrigger.update); gsap.ticker.add(raf); gsap.ticker.lagSmoothing(0); }
  gsap.utils.toArray<HTMLElement>('[data-reveal]').forEach((el) => { if (reduce) return;
    gsap.from(el, { y: 30, autoAlpha: 0, duration: 1, delay: parseFloat(el.dataset.reveal || '0') || 0,
      ease: 'power3.out', scrollTrigger: { trigger: el, start: 'top 88%', once: true } }); });
  gsap.utils.toArray<HTMLElement>('[data-parallax]').forEach((el) => { if (reduce) return;
    const root = el.closest<HTMLElement>('[data-parallax-root]') || el;
    gsap.to(el, { yPercent: (parseFloat(el.dataset.parallax || '0.2') || 0.2) * 100, ease: 'none',
      scrollTrigger: { trigger: root, start: 'top top', end: 'bottom top', scrub: true } }); });
  ScrollTrigger.refresh();
}
function teardown() { ScrollTrigger.getAll().forEach((t) => t.kill()); gsap.ticker.remove(raf); lenis?.destroy(); lenis = null; }
document.addEventListener('astro:page-load', setup);
document.addEventListener('astro:before-swap', teardown);
```
使い方:要素に `data-reveal`(任意で遅延秒 `data-reveal="0.1"`)、ヒーロー背景に `data-parallax="0.18"` ＋親に `data-parallax-root`。
**global.css に Lenis base CSS が必要**(`html.lenis,html.lenis body{height:auto}` ／ `.lenis.lenis-smooth{scroll-behavior:auto!important}`)。
横スクロール領域(カルーセル等)には **`data-lenis-prevent`**。

## カルーセル(Blossom Core)
`Blossom(scroller, options)` は **第2引数必須**(`{}` 可)。スクローラー=ネイティブ scroll-snap の横並び。
```astro
<div id="carousel" class="no-scrollbar flex snap-x snap-mandatory gap-5 overflow-x-auto" data-lenis-prevent>
  { items.map(x => <article class="shrink-0 snap-start" style="flex-basis:min(78vw,380px)">…</article>) }
</div>
<button data-prev>‹</button><button data-next>›</button>
<script>
  import { Blossom } from '@blossom-carousel/core';
  import '@blossom-carousel/core/style.css';
  let b: ReturnType<typeof Blossom> | null = null;
  function init() { const el = document.querySelector<HTMLElement>('#carousel'); if (!el) return;
    b = Blossom(el, { repeat: false }); b.init();
    document.querySelector('[data-prev]')?.addEventListener('click', () => b?.prev({ align: 'start' }));
    document.querySelector('[data-next]')?.addEventListener('click', () => b?.next({ align: 'start' })); }
  document.addEventListener('astro:page-load', init);
  document.addEventListener('astro:before-swap', () => { b?.destroy(); b = null; });
</script>
```
(Swiper/Embla も同じく `astro:page-load` で init / `before-swap` で destroy。)

## プレースホルダ画像(実画像を待たない)
画像レーン完了前は各スロットを**単色ボックス**で代替してレイアウトを確定 → 実画像が来たら `<img>` に差し替え:
```astro
<div class="aspect-[3/4] w-full bg-surface"></div>
```
コンポーネント側を `aspect-[…]` 固定＋`object-cover` にしておけば、実画像差し込みで**シフトしない**。

## 撮影モード(full-page スクショ)
scroll-reveal の隠し要素を写すため、撮影直前に可視化(Playwright `browser_evaluate`):
```js
async () => { document.querySelectorAll('[data-reveal]').forEach(e=>{ e.style.opacity='1'; e.style.visibility='visible'; e.style.transform='none'; });
  if (document.fonts?.ready) await document.fonts.ready; await new Promise(r=>setTimeout(r,500)); }
```

## Tailwind v4 の確実な書き方
- 比率/拡大は**角括弧**で確実に:`aspect-[3/4]` `aspect-[4/5]` `scale-[1.15]`。トークン色は `@theme` から `bg-bg` `text-ink` 等が自動生成。
- **ランタイムで class をトグルする箇所**(ヘッダーのスクロール状態・配色適応など)は **Tailwind ユーティリティを
  JS 文字列で足さない**(スキャン漏れで CSS が出ない)。`is-scrolled`/`over-hero` 等の**意味クラスを1つトグル**し、
  色は **CSS変数**で global.css 側に当てる(例:`[data-header].over-hero:not(.is-scrolled){ --hdr-fg:#fff }`)。
