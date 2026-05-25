+++
title = 'Learning to learn'
date = '2026-05-23T22:25:07-04:00'
weight = 110
draft = false
+++

Understanding something new is only half the work. The other half is applying what you have learned to real-world problems until it becomes instinct. A useful framework for this is *see one, do one, teach one*: first observe how something is done, then do it yourself, then explain it to someone else. Teaching forces you to identify gaps in your own understanding and is one of the most effective ways to solidify knowledge.

## Cramming doesn't work

The brain processes information in two modes. *L-mode* (linear mode) handles logical, sequential thinking: reading code, following steps, writing syntax. *R-mode* (rich mode) handles pattern recognition, insight, and synthesis: the moment a concept suddenly clicks, or you wake up with a solution you couldn't find the night before.

Cramming works against both. Massing all study into a single session overloads L-mode and gives R-mode no time to operate. Sleep and rest are not idle time: they are when the brain consolidates what it encountered during the day. Insight often arrives in the shower or on a walk precisely because you stepped away.

Effective encoding — moving information from short-term to long-term memory — requires three things: attention during initial exposure, spaced repetition over time, and retrieval practice (recalling information from memory rather than re-reading it). Highlighting and re-reading feel productive but produce weak encoding. Writing notes in your own words and testing yourself produce strong encoding.

When you are stuck, stepping away is not procrastination. It is giving R-mode space to work. Return with fresh eyes and the answer is often waiting.

## Skill acquisition

Skill develops in three stages: *imitate, assimilate, innovate*. You begin by copying patterns you observe in books, tutorials, or others' code. As you build experience, you internalize those patterns and apply them without consciously referring back to the original source. Eventually you adapt and combine patterns in novel ways — this is innovation, and it only becomes possible once the fundamentals are automatic.

The *Dreyfus model* describes five stages of expertise in any skill:

```
         ┌─────────────────────┐
         │       Expert        │  Intuitive, fluid, no longer rules-based
         ├─────────────────────┤
         │     Proficient      │  Sees the whole picture; adapts rules to context
         ├─────────────────────┤
         │    Competent        │  Follows rules deliberately; makes deliberate choices
         ├─────────────────────┤
         │     Advanced        │
         │      Beginner       │  Recognizes context; bends rules when needed
         ├─────────────────────┤
         │      Novice         │  Needs rules; can't yet judge when to break them
         └─────────────────────┘
```

Novices need concrete rules and step-by-step guidance. They cannot yet judge which rules to apply in a given context. Advanced beginners start recognizing patterns across situations but still rely heavily on guidance. Competent practitioners can troubleshoot and plan; they make deliberate choices and can prioritize. Proficient practitioners see the whole picture and can identify when standard approaches fall short. Experts operate intuitively — they recognize situations and respond without consciously working through rules.

Two practical implications follow. First, a novice and an expert need completely different kinds of help: rules and examples for the novice, context and edge cases for the expert. Giving a novice the expert's perspective too early creates confusion. Second, moving between stages requires time on task — there is no shortcut past competence to proficiency. Deliberate practice at the edge of your current ability is what drives progression.

## Learning habits

Good habits remove the decision overhead from learning. When you do not have to decide whether to study, you study. A few practices make a measurable difference.

**Exploration over consumption.** Read widely across domains, not just within your specialty. Ideas from distributed systems appear in database design. Ideas from cognitive science apply to API design. Keep an active list of things you want to explore: papers, tools, talks, books. Work through it slowly rather than hoarding it.

**Spaced repetition.** Review material at increasing intervals: one day after first encounter, then three days, then a week, then a month. Each successful recall strengthens the memory trace. Flashcard tools like Anki implement this schedule automatically, but a simple notebook works too.

**The Learning Depth Strategy.** Christopher Judd describes a four-level approach to any new technology:

1. *Awareness* — know the technology exists and what problem it solves
2. *Conceptual understanding* — understand how it works without necessarily being able to use it
3. *Practical skill* — be able to use it to solve a real problem
4. *Mastery* — know its limits, edge cases, and tradeoffs well enough to teach it

Not everything needs to reach level four. The strategy is useful because it makes the decision explicit: for this skill, what level do I actually need?

**Writing as thinking.** Writing forces precision. If you cannot explain something in writing, you do not fully understand it. Maintain notes you will actually reread — not comprehensive transcripts, but your own synthesis of what mattered and why.

**Timeboxing exploration.** Unstructured exploration is valuable but can expand to fill all available time. Set a timer. Thirty minutes of focused exploration followed by deliberate rest is more effective than three hours of unfocused browsing.

## Deliberate skill acquisition

Deliberate practice is not the same as experience. Spending ten years doing the same task repeatedly produces competence at that task, not growth beyond it. Deliberate practice means working at the edge of your current ability, with immediate feedback, on skills you have specifically identified as weak.

### Fundamentals first

The instinct when learning a new language or framework is to reach for abstractions quickly: ORMs, frameworks, CLI generators. Resist it. Understand what the abstraction replaces before you use it. A developer who understands SQL writes better ORM queries. A developer who understands HTTP writes better REST clients. Fundamentals compose. Abstractions leak.

The core skills that transfer across every language and domain:

- **Data structures and algorithms:** slices and maps, `container/heap`, `sort.Slice`, binary search, Big O analysis
- **Design patterns:** functional options, interface composition, middleware chains, worker pools, error wrapping
- **Testing methodologies:** table-driven tests, subtests with `t.Run`, benchmarks with `testing.B`, `httptest`, race detector
- **Version control:** branching strategies, semantic commits, conflict resolution, `go mod tidy`, module versioning
- **Database design:** `database/sql` and connection pooling, prepared statements, transactions, schema migrations, query optimization

### Build things you do not need

Tutorials produce tutorial knowledge. Building something from scratch — even something small and useless — forces decisions that tutorials make for you. Write a small HTTP server before using a framework. Parse a file format by hand before using a library. The friction is the learning.

### T-shaped development

A *T-shaped developer* has broad awareness across many areas (the horizontal bar) and deep expertise in one or two (the vertical bar). Breadth lets you recognize when a problem calls for a tool or approach outside your specialty. Depth lets you execute reliably and mentor others. Both matter. Depth without breadth produces blind spots; breadth without depth produces a generalist who cannot ship.

Build depth deliberately by choosing one area — a language, a system, a domain — and staying with it long enough to reach proficiency. Build breadth by following curiosity: read release notes for tools you do not use, attend talks outside your area, pair with engineers from other teams. The horizontal bar grows naturally when you stay curious; the vertical bar requires intention.
