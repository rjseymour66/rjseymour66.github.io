+++
title = 'Software Development'
date = '2026-05-09T08:29:11-04:00'
weight = 100
draft = false
+++

## The lazy programmer ethos

The lazy programmer ethos is a philosophy built on efficiency. Strategic laziness isn't about avoiding work. It's about choosing the right work.

When you resist the urge to immediately start coding, you create space to think. That thinking time pays off. You write less code, avoid dead ends, and build solutions that hold up over time.

In practice, the lazy programmer:

- Automates repetitive tasks rather than doing them manually
- Reuses existing code before writing new code
- Asks "do I actually need this?" before starting any task
- Writes simple code that's easy to change, not code that anticipates every future requirement

The goal isn't to do less. It's to eliminate unnecessary work so you can focus on what matters.

## Shift left

"Shifting left" means moving testing and quality checks earlier in the development cycle. The name comes from project timelines where earlier stages appear on the left. The further right a bug travels, the more expensive it becomes to fix.

In practice, shifting left means:

- Write tests before or alongside your code, not after
- Run linters and static analysis tools on every commit
- Review code early and often, not just before release
- Include security scanning in your CI pipeline from the start
- Define acceptance criteria before writing a single line of code

A bug caught in development costs minutes to fix. The same bug caught in production can cost hours, damage users, and erode trust. Shifting left keeps quality work close to where the code is written.

## The five whys

The five whys is a problem-solving method for finding the root cause of an issue. Instead of treating symptoms, you ask "why" five times, each time directing the question at the previous answer. By the fifth why, you've usually uncovered the underlying cause.

**Example:**

1. Why did the site go down? The server ran out of memory.
2. Why did the server run out of memory? A process leaked memory over time.
3. Why did the process leak memory? The connection pool was never closed after use.
4. Why was the connection pool never closed? The error-handling path skipped the cleanup code.
5. Why did the error-handling path skip cleanup? There were no tests covering the error path.

The root cause isn't the outage or even the leak. It's a gap in test coverage. Fixing only the symptom (restarting the server) guarantees the problem returns.

Use the five whys when a bug, incident, or failure keeps recurring. The method forces you past the obvious fix and toward the change that actually prevents recurrence.
