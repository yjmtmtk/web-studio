# web-studio

A from-scratch web-production skill built on the stack AI is strongest at:
**Astro + Tailwind + typed `.astro` components** (Props = enforced contracts) +
design tokens, with photography from **unsplash-fetch**.

Its headline feature is a **visual self-critique loop**: build → screenshot →
multimodal critique against a design rubric → targeted fixes → rebuild → repeat.
That closes the feedback loop a one-shot generation lacks, and is the biggest
quality lever. Rich by default (scroll animation, carousels, reveal), with
White Space layout discipline.

## Install

```bash
npx skills add yjmtmtk/web-studio
```

## Requirements

- **Node** (`npm create astro`, `npx astro build`).
- **[unsplash-fetch](https://github.com/yjmtmtk/unsplash-fetch)** (`npx unsplash-fetch`, zero-install) + `UNSPLASH_ACCESS_KEY` for photography.

## The loop in one line

```bash
npm create astro@latest site -- --template minimal --install --typescript strict --yes
cd site && npx astro add tailwind --yes
# author typed components → npx astro build → screenshot → critique → fix → repeat
```

Most of the method lives in `SKILL.md`. License: MIT.
