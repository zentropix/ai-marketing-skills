---

## Preamble (runs on skill start)

```bash
# Version check (silent if up to date)
python3 telemetry/version_check.py 2>/dev/null || true

# Telemetry opt-in (first run only, then remembers your choice)
python3 telemetry/telemetry_init.py 2>/dev/null || true
```

> **Privacy:** This skill logs usage locally to `~/.ai-marketing-skills/analytics/`. Remote telemetry is opt-in only. No code, file paths, or repo content is ever collected. See `telemetry/README.md`.

---
name: design-gauntlet
description: Run a gauntlet loop on any design website or landing page — pick a named best-in-class page as the quality bar, split the page into judgeable pieces (hero, type, color, motion, mobile, CTA), fan out builder + harsh-critic agent pairs per piece, compare screenshots blind against the bar, and loop until the critic picks ours. Pairs taste (blind A/B vs a real page) with numbers (CRO audit score, Lighthouse). Use when asked to "gauntlet" a page, "loop until it beats [site]", "make this landing page as good as [brand]", "design gauntlet", or to iterate a page design against a specific reference until it wins.
---

# Design Gauntlet

Run a builder-vs-critic gauntlet loop on a design website or landing page until it beats a real, named reference page in a blind comparison.

> The gauntlet loop technique is [Matt Shumer's](https://github.com/mshumer) (from [Claude of Duty](https://github.com/mshumer/Claude-of-Duty)); the reusable skill packaging is by [RoboNuggets](https://github.com/robonuggets/gauntlet-loop) (CC BY 4.0). This skill specializes it for design sites and landing pages and wires it into this repo's CRO tooling.

## The core idea

Agent-built pages stop at "good enough" because nothing holds them to a standard. A rubric doesn't work — the agent grades itself against words it wrote, and scores drift upward every round. A **bar** works: a specific, live, undeniably good page the critic can screenshot and put next to ours with the labels stripped. The critic's job is binary — *which one is better?* — and the loop only exits when ours wins.

For design pages, the bar has one extra advantage: it's trivially fetchable. Any live page can be screenshotted at the same viewports and compared side by side.

## Flow

### 1. Set the bar (do this first, always)

If the user named a reference, use it. Otherwise offer **2–3 candidate bars** — one line each — and **stop and wait for their pick**. Do not start building.

Every bar must pass three tests:

- **Named.** "Stripe's pricing page", not "great SaaS pricing pages".
- **Fetchable.** The critic can open it in a browser and screenshot it at desktop (1440px) and mobile (390px). If it's paywalled, geo-blocked, or gone, it is not a bar.
- **Comparable.** Same kind of page. A landing page competes against a landing page, a pricing page against a pricing page.

Prefer the hardest bar the agent can genuinely reach. A bar that's too easy exits on round one and produces mediocrity with a trophy.

Good bars by page type:

| Page type | Bar that works |
|---|---|
| SaaS landing page | Linear, Stripe, Vercel — the live homepage, both viewports |
| Ecom / DTC product page | A named brand's current product page (Nike, On, Gymshark, Apple) |
| Lead-gen / offer page | A named competitor's live funnel page, screenshotted through the flow |
| Pricing page | Stripe or a named category leader's pricing page |
| Agency / portfolio site | A named Awwwards Site of the Day (the specific site, not the award) |

### 2. Attach the measurable half

Taste plus a number beats taste alone. Alongside the blind A/B, name at least one metric both pages get scored on:

- **CRO score** — `python3 conversion-ops/cro_audit.py --url <ours> ` vs the same on the bar. 8 conversion dimensions, no headless browser needed.
- **Performance** — Lighthouse performance score or LCP at mobile, ours vs the bar's.
- **Copy strength** (optional) — run hero headline / CTA variants through `autoresearch` and only ship 85+ scorers.

The critic's verdict is: ours must **win the blind visual pick AND not lose the numbers**. If the visual pick is ours but the CRO score dropped, that round doesn't count as a win.

### 3. Fetch the real bar

Before any building: screenshot the bar page at 1440px and 390px, and save them to `docs/design-references/bar/`. Extract its design tokens (fonts, colors, spacing rhythm) the way `clone-site` does in recon — `getComputedStyle()` on live elements, not guesses from the screenshot. The critic compares against the real captures, never against a description of them.

Requires Chrome MCP (`claude --chrome`) or Playwright for the captures.

### 4. Split the page into judgeable pieces

Break the page into the smallest pieces that can be improved and judged on their own. For a landing page the standard split is:

- **Hero** — first-viewport impression, headline, primary CTA
- **Typography** — scale, hierarchy, line lengths, rhythm
- **Color & contrast** — palette discipline, dark/light handling
- **Imagery & assets** — quality, consistency, art direction
- **Motion & interaction** — scroll behavior, hovers, micro-interactions
- **Mobile** — the 390px experience judged on its own, not as a shrunken desktop
- **CTA & conversion path** — above-fold clarity, friction, trust signals

Each piece gets its own builder/critic pair. Don't pre-assign an architecture or component layout beyond this — the builders decide, and they decide better than a spec written before the work started.

### 5. Run builder + critic pairs

For each piece, fan out **two separate agents**:

- **Builder** — improves that piece of our page. Full context on the goal and prior critic feedback for its piece only.
- **Critic** — fresh context every round. It must not know how hard the builder tried or what changed. It opens the actual rendered output in a browser, screenshots it at the same viewports as the bar, puts the two screenshots side by side **with labels stripped** (name files `A.png` / `B.png`, randomize which is which), and answers exactly two things:
  1. Which one is better — A or B? (a pick, never a score out of 10)
  2. The single biggest remaining gap in the loser.

The gap goes back to the builder. That's one round.

In Claude Code, run pairs as parallel subagents (`ultracode` / Workflow fan-out) and `/loop` each piece. On any other agent: "run builders and critics as parallel subagents; keep looping until the critic picks ours blind."

### 6. Exit condition

A piece is done when the critic picks ours blind. The page is done when every piece has won **and** a final whole-page critic — fresh context, both full-page screenshots, labels stripped — picks ours at both viewports, and the numbers from step 2 hold. Never exit after a fixed round count. The only other exit is the user stopping the run.

### 7. Keep a live progress page

Maintain a simple progress artifact (HTML or markdown) updating as rounds complete: per-piece status (round count, last verdict, biggest gap), current full-page screenshot vs the bar, current CRO/Lighthouse numbers. The user watches this instead of the transcript.

## Paste-ready prompt template

When the user just wants the prompt (to run in a fresh session), fill this in — around 120–180 words, plain sentences, no headings:

```
Build [PAGE — audience, vibe, offer, non-negotiables].

The bar is [NAMED PAGE + URL]. Screenshot it at 1440px and 390px first and compare against those captures directly, not against a description of them. Also score both pages with conversion-ops/cro_audit.py — ours has to win the blind pick without losing the CRO score.

Break the page into the smallest pieces that can be improved and judged on their own — hero, type, color, imagery, motion, mobile, CTA. For each piece, fan out a builder and a separate critic with fresh context. The critic screenshots our rendered output, puts it next to the bar blind with the labels stripped, says which is better, and names the single biggest remaining gap. Then it goes back to the builder.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

Add a budget/cost ceiling line only if the user named one. Add deploy-target or image-generation tool names only if the goal needs them. Leave stack, file layout, and round counts out.

## Pairs with

- **`clone-site`** — recon methodology for capturing the bar (screenshots, `getComputedStyle()` tokens, asset extraction), and the Next.js build pipeline if starting from scratch.
- **`conversion-ops`** — `cro_audit.py` as the measurable half of every verdict.
- **`autoresearch`** — 50-variant expert-panel optimization for the copy inside the winning design (headlines, CTAs, hero text).

## What breaks the loop

- **A vague bar.** The critic invents the comparison and approves everything. The most common failure by far — refuse to start without a named, fetchable page.
- **The builder judging its own work.** The critic must be a separate agent with fresh context every round.
- **A soft critic.** Binary pick only. Scores out of 10 drift upward every round.
- **Comparing against a description.** The critic must open real screenshots of the real bar, captured this run.
- **A fixed round count.** The exit is winning, or the user calling it.
- **Winning taste while losing the numbers.** A prettier page with a worse conversion path is a loss — that's why the CRO score rides along.

## Ethics note

The bar is a quality reference, not a source. Beat the reference page — don't reproduce its copy, logos, imagery, or trade dress. For literal replication of a page you own or have rights to, use `clone-site`.
