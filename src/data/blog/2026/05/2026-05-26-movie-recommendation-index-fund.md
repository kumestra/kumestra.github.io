---
title: The Index Fund Model for Movie Recommendations
description: Why "must-watch classics" lists fail ordinary viewers, and how aggregating personal top-10s by decade produces a recommendation system that maximizes expected enjoyment — structured like an S&P 500 plus sector ETFs.
pubDatetime: 2026-05-26T02:10:00Z
tags: [film, recommendation-systems, culture, design]
---

Most "best films ever made" lists serve the wrong goal. They answer "what was most important to the history of cinema?" not "what should I actually watch tonight?" These are different questions, and confusing them produces lists that disappoint ordinary viewers while feeling obligatory to engage with.

## Three Things That Don't Overlap

Every film occupies different positions on three independent axes:

| Axis | Question | Who it serves |
|------|----------|--------------|
| **Worth experiencing** | Does it give viewers genuine pleasure right now? | Ordinary viewers |
| **Worth studying** | Does understanding it help you read the whole medium? | Students of film |
| **Historically significant** | Did it influence everything that came after? | Historians |

The problem: almost every "must-watch" list conflates these three. Ordinary viewers — people who just want to spend two good hours watching something — care only about the first axis. But canonical lists like *Sight & Sound*'s Greatest Films poll are built on the second and third.

## The Citizen Kane Case

*Citizen Kane* is the clearest example of this collision. Its reputation rests on deep-focus cinematography, low-angle shots, non-linear structure, and unreliable-narrator puzzles — all genuinely revolutionary in 1941. But here's the problem: a viewer in 2026 has already seen hundreds of films that absorbed those innovations. To feel Kane's original impact, you'd need to watch thirty-odd pre-1941 films first, just to register what was being broken.

Compare this to the first AJAX implementation. That code was revolutionary — it proved dynamic web applications were possible. But no one would point a developer at it today and say "you should study this to understand how to build web apps." The techniques it pioneered have been executed far better by everything that followed. The code retains historical value; it has no practical value.

Kane's situation is structurally similar. *Pulp Fiction*, *Memento*, and *Rashomon* each execute non-linear narrative more cleanly, more accessibly, or more philosophically than Kane does — for a viewer who has never seen Kane and has no context for what Kane invented. If the goal is maximizing a viewer's enjoyment of non-linear storytelling, recommending Kane requires pre-loading a context that most viewers don't have and don't need.

This points to a useful filter for any canonical work:

> If a film's value lies primarily in "it originated X" and X has since been executed better by later films, it belongs on a film history syllabus — not on a list for general audiences.

## Why Existing Lists Fail

**Sight & Sound** is the most respected critical list. Its mechanism — asking working directors and critics to submit top-10 ballots every decade — is sound. The flaw is the respondent pool. Critics and filmmakers share a professional formation that normalizes certain reference points, rewards formal complexity, and carries historical context as background knowledge. Their list is a reliable map of insider consensus, not of what an ordinary viewer would enjoy.

**IMDb Top 250** runs on raw democratic voting. The mechanism sounds appealing — pure audience preference — but it has a serious volatility problem. When *Avengers: Endgame* opened, its score spiked into the top 10 on a wave of immediate emotional response. These pulse effects are noisy: they measure theatrical excitement, not durable appreciation. Films also "stick" on the list past their warranted position because inertia favors incumbents. The result conflates hype with quality.

Neither list answers: *what films, evaluated on their own terms by people without specialist training, have genuinely held up over time as good viewing experiences?*

## A Better Mechanism: Aggregate Personal Top-10s

The proposal: survey a large sample of general viewers and ask each person one question — "list the ten best films you have ever seen."

This design has several structural advantages over existing approaches:

**It captures cold recall, not hot reaction.** A film earns a spot on your personal top-10 only after the initial excitement has faded. Someone who puts *Endgame* on their list a week after opening weekend might remove it two years later when the adrenaline wears off. The films that survive this natural cooling are the ones that genuinely stuck. IMDb measures emotional state at time of rating; this mechanism measures durable memory.

**It embeds the recommendation instinct.** When people curate a personal top-10, they're already implicitly asking "what would I tell someone else to watch?" The question doesn't need to be posed directly — it's baked into the act of selection.

**Older films losing ground is a feature, not a bug.** If an ordinary viewer hasn't seen pre-1950 films (and most haven't), those films can't appear on their lists regardless of historical importance. This looks like a flaw. It isn't. The list's purpose is to maximize expected enjoyment for viewers as they actually exist — not to provide historical fairness. If *Pulp Fiction* crowds out *Citizen Kane* on a non-linear-narrative dimension, that's the mechanism working correctly.

**The time-decay curve is built in.** Over a rolling survey, any film that enters on hype will gradually fall off lists as people reassess. Films that hold their position for a decade or more have genuinely earned it. This is a quality-verification mechanism that doesn't require editorial intervention.

## The Index Fund Architecture

The mechanism maps cleanly onto index fund logic:

```
Global aggregate  →  S&P 500 equivalent
  All respondents, all films, ranked by decade

Genre cuts        →  Sector ETFs
  "Best romance films by decade"
  "Best sci-fi films by decade"

Demographic cuts  →  Factor ETFs
  Results segmented by age group
  Results segmented by gender or region
```

The underlying data is collected once. Every segmented list is derived from the same dataset by applying different filters — no separate surveys needed for each genre or demographic. A viewer who wants a general starting point uses the global list. A viewer who specifically wants great horror films pulls the horror segment.

This solves a problem that both Sight & Sound and IMDb ignore: the single-ranking assumption. A global consensus ranking is always a compromise that over-represents the median taste profile. Genre and demographic slices let viewers find the list that was optimized for someone like them.

One practical constraint: the deeper you cut, the smaller the sample. A global ranking over 10,000 respondents is statistically stable. A "best horror films of the 1980s" segment drawn from the same survey might rest on 800 responses. Sub-lists need either lower update frequency or a much larger base survey — the same liquidity trade-off that makes small-cap ETFs less reliable than the S&P.

## The Transparency Advantage Over Netflix

This system is structurally similar to what Netflix already does with personalized recommendations. Netflix also infers "films people like you have liked" and uses that to drive suggestions. The difference is opacity: Netflix's model is private, personalized, and unauditable. You can't compare your recommendations to someone else's. You can't see what the aggregate actually values or argue with it.

An aggregated public list built on personal top-10s is transparent and discussable. The list makes an explicit claim — *these are the films that have held up in ordinary people's memories across time* — and that claim can be evaluated, contested, and improved. That's a more honest relationship with viewers than a black-box recommendation feed, even if it trades away the precision of individual personalization.

## What This Would Look Like in Practice

**Letterboxd's decade rankings** are the closest existing approximation. The respondent pool — serious but non-professional film enthusiasts — sits between IMDb's mass audience and Sight & Sound's insiders. Letterboxd has some resistance to rating spikes because its heavy-user base is large and new releases have natural cooling periods. Looking at its decade-by-decade highest-rated films, you get things like *Parasite*, *Whiplash*, *Spirited Away*, *The Godfather* — films that are both well-made and genuinely enjoyable to watch without requiring specialist context. *Citizen Kane* doesn't appear.

The remaining gap is that Letterboxd doesn't publish a formal "top-10 per decade" constructed from aggregated personal favorites — its rankings are based on star ratings rather than curation lists. Building the mechanism described here would require asking Letterboxd users to submit ranked personal top-10s, then aggregating those separately from the star-rating system.

The data exists. The framework is clear. What's missing is someone willing to publish the result without insisting it carry the authority of a *Sight & Sound* poll — because the legitimacy of this list doesn't come from institutional backing. It comes from the mechanism itself.

## Summary

The argument from *Citizen Kane* down to this system is a single logical chain:

1. A film's historical importance doesn't predict whether an ordinary viewer will enjoy watching it.
2. Current canonical lists conflate importance with watchability, producing recommendations that serve film historians more than general audiences.
3. A mechanism that aggregates personal top-10s from general viewers, updated over time, naturally filters for durable enjoyment over historical prestige.
4. That global list plus genre/demographic segments functions like an index fund family: a diversified default for everyone, with sector ETFs for viewers who know their own tastes.
5. The films that stay on the list without institutional support are the ones worth watching. The rest are worth knowing about — which is a different thing.
