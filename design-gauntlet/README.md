# ⚔️ Design Gauntlet

**Loop a landing page against a real best-in-class site until it wins a blind comparison.**

Most agent-built pages stop at "good enough" because nothing holds them to a standard. This skill gives your agent a standard it can't argue with: a **named, live, screenshot-able page** — Linear's homepage, Nike's campaign page, a competitor's funnel — and a harsh critic that puts your page next to it *blind* and picks a winner. The loop doesn't exit until yours is the pick.

> **Credit:** The gauntlet loop technique is [Matt Shumer's](https://github.com/mshumer), from [Claude of Duty](https://github.com/mshumer/Claude-of-Duty). The reusable skill packaging is [RoboNuggets' gauntlet-loop](https://github.com/robonuggets/gauntlet-loop) (CC BY 4.0). This skill specializes the pattern for design websites and landing pages and wires it into this repo's CRO tooling.

---

## What It Does

```
Input:  "Make my SaaS landing page as good as Linear's — gauntlet it"
Output: A page that beat Linear's homepage in a blind screenshot A/B,
        without losing on CRO score or performance
```

### The Loop

1. **Set the bar** — a specific named page (never a category like "award-winning SaaS sites"). Offered as 2–3 candidates if you didn't name one.
2. **Fetch the real thing** — screenshot the bar at desktop + mobile, extract its actual design tokens. The critic compares against real captures, never a description.
3. **Attach a number** — CRO audit score (via [`conversion-ops`](../conversion-ops/)) and/or Lighthouse, ours vs. the bar's. Taste plus a number beats taste alone.
4. **Split the page** — hero, typography, color, imagery, motion, mobile, CTA. Each piece gets its own builder/critic pair.
5. **Builder vs. harsh critic** — the critic has fresh context every round, sees both screenshots with labels stripped, and gives a binary pick plus the single biggest remaining gap. No scores out of 10 (they drift upward every round).
6. **Exit only on a win** — every piece picked blind, then a final whole-page blind pick at both viewports, with the numbers holding. Never a fixed round count.

## Why a Bar and Not a Rubric

A rubric asks the agent to grade itself against words it wrote. A bar makes it compare against something that already exists and is undeniably good. For design pages the bar is trivially fetchable — any live page can be screenshotted at the same viewports and judged side by side.

## Works With

| Skill | Role |
|---|---|
| [`clone-site`](../clone-site/) | Recon methodology for capturing the bar (screenshots, computed-style tokens) and a Next.js build pipeline |
| [`conversion-ops`](../conversion-ops/) | `cro_audit.py` — the measurable half of every verdict |
| [`autoresearch`](../autoresearch/) | Expert-panel optimization of the copy inside the winning design |

## Requirements

- Chrome MCP (`claude --chrome`) or Playwright, for screenshotting the bar and your rendered page
- Python 3.10+ if using the CRO audit as the measurable half
- Works in Claude Code (`/loop` + subagent fan-out) or any agent that can run parallel subagents

## Quick Start

```
/design-gauntlet my lead-gen landing page, dark and premium, has to beat [competitor URL]
```

Or ask for just the paste-ready prompt and run it in a fresh session — the template is in [SKILL.md](./SKILL.md).

## Structure

```
design-gauntlet/
├── README.md     # This file
└── SKILL.md      # The full skill — flow, bar tests, prompt template
```

---

## License

Skill text CC BY 4.0 (inherited from [gauntlet-loop](https://github.com/robonuggets/gauntlet-loop)); repo tooling MIT — see [LICENSE](../LICENSE).


---

<div align="center">

**🧠 [Want these built and managed for you? →](https://singlebrain.com/?utm_source=github&utm_medium=skill_repo&utm_campaign=ai_marketing_skills)**

*This is how we build agents at [Single Brain](https://singlebrain.com/?utm_source=github&utm_medium=skill_repo&utm_campaign=ai_marketing_skills) for our clients.*

[Single Grain](https://www.singlegrain.com/?utm_source=github&utm_medium=skill_repo&utm_campaign=ai_marketing_skills) · our marketing agency

📬 **[Level up your marketing with 14,000+ marketers and founders →](https://levelingup.beehiiv.com/subscribe)** *(free)*

</div>
