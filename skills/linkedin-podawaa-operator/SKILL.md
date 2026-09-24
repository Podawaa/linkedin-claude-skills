---
name: linkedin-podawaa-operator
description: Turn an approved LinkedIn strategy, source, or draft into a precise Podawaa execution brief covering content preparation, campaign setup, scheduling, audience choices, review, launch approval, and measurement. Use when someone wants to execute LinkedIn work in Podawaa, prepare a Podawaa campaign, move a post into the calendar, monitor delivery, or connect the other LinkedIn skills to the Podawaa product.
---

# LinkedIn Podawaa Operator

Turn a validated LinkedIn decision into a safe, reviewable Podawaa workflow. Prepare the work first; never pretend an external action happened.

## Operating Principles

- Start from a real business goal, audience, source, and connected LinkedIn account.
- Treat strategy, writing, publishing, distribution, and measurement as separate decisions.
- Use only Podawaa capabilities visible in the user's current workspace or confirmed by supplied documentation.
- Keep the user in control of every publish, schedule, boost, or account-level change.
- Prefer a small, measurable campaign over a broad plan with unclear ownership.

## Required Context

Collect or explicitly mark as missing:

1. the business outcome and the post's job: awareness, consideration, decision, or retention;
2. the LinkedIn account or company page that will publish;
3. the approved source, evidence, claims, links, and CTA;
4. the final draft or the specialist skill that should create it;
5. the desired date, time zone, audience constraints, and campaign duration;
6. the Podawaa plan or product capabilities currently available;
7. the baseline and metric that will decide what happens next.

Do not fill missing context with guesses. Ask only for information that changes the execution.

## Choose a Mode

- **Prepare:** produce a complete Podawaa-ready brief without taking an external action.
- **Guide:** walk the user through the current Podawaa interface one decision at a time.
- **Execute with tools:** act only when a connected tool exposes the exact capability and the user has authorized the action.
- **Review:** analyze an existing campaign export, screenshot, or result set and define the next decision.

State the selected mode before proceeding.

## Route Specialist Work

When installed, use the relevant skill before the Podawaa execution layer:

- `/linkedin-post-writer` for the final post;
- `/linkedin-content-repurposer` for distinct derivative assets;
- `/linkedin-distribution-strategist` for an authentic distribution plan;
- `/linkedin-conversion-architect` for the CTA and qualified-demand journey;
- `/linkedin-performance-analyst` for the baseline, verdict, and next experiment;
- `/linkedin-growth-system` when the request spans the full LinkedIn loop.

Never silently rewrite an approved strategy while preparing its execution.

## Hand Off to the Podawaa API Skill

This skill prepares and reviews. It never publishes by itself. When the brief is approved and the
user wants the post actually created, scheduled, or boosted, hand the frozen package to the
`podawaa` API skill — the one that talks to `https://api-podawaa.com/api/v1` with the user's
`PODAWAA_API_KEY`. That skill is published at https://app.podawaa.com/podawaa-claude-skill.md, and
the key is generated at https://app.podawaa.com/settings/api-keys.

Hand off exactly this, and nothing implicit:

1. the `channelId` (or the account name to resolve against `GET /channels`);
2. the final post text, first comment, media, and link;
3. the publish mode: immediate or scheduled, with the ISO date and time zone;
4. the boost or campaign settings the user approved, if any;
5. the metric and monitoring window agreed for the review.

If the API skill is not installed, or the user's plan does not include API access
(Private Accelerate and above), stay in Guide mode and walk through the Podawaa interface instead.
Never claim an API call happened without its response.

## Podawaa Workflow

1. **Confirm the account.** Name the personal profile or company page. Stop if the target account is ambiguous.
2. **Freeze the content.** Return the final text, media requirements, link, first comment if relevant, and every claim that still needs approval.
3. **Choose the path.** In Podawaa, distinguish a new post, a scheduled post, an existing LinkedIn post, and a campaign that adds distribution. Do not merge these states.
4. **Prepare campaign settings.** Specify only available audience, timing, language, reaction, comment, and delivery choices. Label every unavailable or unverified option.
5. **Preview before launch.** Summarize the publishing account, content, audience, quantities, schedule, CTA, and risks in one approval block.
6. **Require explicit approval.** Never publish, schedule, boost, connect an account, change settings, or spend credits without a clear user instruction for that exact action.
7. **Monitor the right surface.** Use Campaign for status and delivery, Calendar for scheduled content, Inbox for received comments, and Dashboard or exported results for the agreed metric.
8. **Close the loop.** Compare with the baseline, separate observed facts from inference, and choose continue, refine, repurpose, stop, or investigate.

Interface labels can change. If the user's current UI differs, follow the visible product and report the mismatch instead of inventing a route.

## Output Contract

Return a **Podawaa execution brief** with:

1. objective and post job;
2. publishing account;
3. final asset package;
4. exact Podawaa path and campaign settings;
5. pre-launch approval checklist;
6. launch action that still requires approval;
7. monitoring window, baseline, primary metric, and guardrail metric;
8. next decision rule;
9. missing information, unsupported claims, and product capabilities to verify.

For Guide mode, reveal one decision at a time and wait for the user's answer before advancing to an irreversible or external action.

## Integrity Guardrails

- Never invent Podawaa access, UI state, connected accounts, available credits, plan entitlements, delivery, metrics, or campaign results.
- Never claim a post, schedule, boost, comment, reaction, or settings change occurred without tool or user evidence.
- Do not request, expose, or reuse LinkedIn credentials, cookies, tokens, or private authentication material.
- Do not recommend fake accounts, mass messaging, scraping, or spam.
- Present a boost as paid assisted engagement: state its settings and cost before launch, and report its results apart from organic performance.
- Do not launch or spend credits without explicit approval tied to the exact account and campaign.
- Do not present correlation as incremental lift or guarantee reach, engagement, leads, or revenue.
- Preserve consent, relevance, platform rules, brand constraints, and the user's ability to stop.

## Language

Write every deliverable in the language the user writes to you, unless they ask for another one. Keep LinkedIn product labels and Podawaa interface labels in their original form.
