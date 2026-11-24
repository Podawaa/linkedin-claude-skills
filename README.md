# LinkedIn skills for Claude Code

Fifteen [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for planning, writing, distributing and measuring LinkedIn content, and for prospecting without automating LinkedIn.

Ten of them are pure method: they need no account, no API key and no network access. They work on the evidence you give them (interview notes, drafts, post metrics) and refuse to invent proof, quotes or results.

Four prospecting skills read LinkedIn through a browser tool such as Claude in Chrome when one is available, or work from text you paste. They are read-only by design: they never send an invitation, a message or a comment, because LinkedIn prohibits browser extensions that automate activity. They prepare the work; you click.

The last one, `podawaa`, publishes, schedules and boosts posts through the [Podawaa](https://podawaa.com) API and requires a paid Podawaa plan.

## Install

As a plugin:

```
/plugin marketplace add Podawaa/linkedin-claude-skills
/plugin install podawaa-linkedin@podawaa
```

Or copy the folders you want into `~/.claude/skills/` (all projects) or `.claude/skills/` (one project):

```bash
git clone https://github.com/Podawaa/linkedin-claude-skills
cp -r linkedin-claude-skills/skills/* ~/.claude/skills/
```

## Skills

| Skill | What it does |
|---|---|
| `linkedin-growth-system` | Orchestrates the full loop below and routes each phase to the matching skill. Start here for a 30- or 90-day plan or an account audit. |
| `linkedin-audience-researcher` | Turns interview notes, sales calls, support tickets or surveys into an evidence table, problem clusters and content opportunities. |
| `linkedin-positioning-architect` | Profile audit, positioning sentence, headline and About rewrite, signature beliefs and proof gaps. |
| `linkedin-content-strategist` | Pillars, signature series, a four-week calendar, a scorecard and one controlled experiment. |
| `linkedin-post-writer` | Drafts or rewrites a post in the author's voice, with three distinct openings and a list of claims still to verify. |
| `linkedin-content-repurposer` | Turns one source (article, webinar, podcast, report) into distinct assets, and discards variants that add nothing new. |
| `linkedin-distribution-strategist` | A 72-hour distribution plan, advocacy briefs and a channel map. |
| `linkedin-conversion-architect` | CTA ladder, profile-to-offer journey, qualification rules and an attribution plan with explicit blind spots. |
| `linkedin-performance-analyst` | Compares posts against a same-account baseline, separates measured from inferred, and states what the data does not prove. |
| `linkedin-podawaa-operator` | Turns an approved plan into a reviewable Podawaa execution brief and hands it to the `podawaa` skill. |
| `linkedin-prospect-signals` | Finds prospects by recent activity the search filters cannot express (substantive comments, company reshares, competitor engagement) and returns a short reasoned list. |
| `linkedin-followup-tracker` | Builds a follow-up queue from recent connections and conversations (new, day 7, 14, 60) with the last-exchange context and a draft for each. |
| `linkedin-acceptance-rate` | Rebuilds the invitation acceptance rate over a window, splits it by segment and note, and names the likely bottleneck. |
| `linkedin-comment-targets` | Picks a few posts your prospects are already reading where you have something specific to add, with an angle for each. You write the comment. |
| `podawaa` | Lists LinkedIn channels, publishes or schedules posts (first comment, PDF carousel), boosts posts already live on LinkedIn, and polls post status through the Podawaa REST API. Needs `PODAWAA_API_KEY` and a Private Accelerate or Dominate plan. |

Every skill answers in the language you write to it.

## Usage

```
/linkedin-post-writer Turn these notes into a post: ...
/linkedin-performance-analyst Here are my last 12 posts with impressions and replies. What should I repeat?
/linkedin-growth-system Build a 30-day plan. I sell compliance audits to mid-size fintechs.
/linkedin-prospect-signals Find 20 heads of finance who commented on posts about month-end close in the last 90 days.
```

## License

MIT
