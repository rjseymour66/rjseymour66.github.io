+++
title = 'Reading Code'
date = '2026-05-09T08:39:39-04:00'
weight = 20
draft = false
+++

## Brownfield development

Much of your time as a software engineer is spent in brownfield development. You work within the limits of an existing codebase, dealing with legacy code rather than starting from scratch. Brownfield work comes with four core problems to solve.

Understand the business problem
: Before changing anything, understand why the feature or fix exists. Code without business context leads to solutions that are technically correct but practically wrong.

See the problem through the eyes of the previous developer
: The code you inherit made sense to someone at a specific point in time. Before judging a decision as wrong, find out what constraints the original developer was working under. Context turns what looks like a mistake into a reasonable tradeoff.
  
  Version control is your primary tool here. Use `git blame` to see who wrote a line and when. Use `git log` to read the commit messages that explain why a change was made. PR comments and code review threads often contain the real decision rationale. Look there before assuming a choice was arbitrary.

Identify the right level of abstraction
: Existing code often lacks the abstraction needed to make changes safely. It may be too tightly coupled, or it may have abstracted the wrong things. Before adding features, assess whether the structure can support them.

Account for technical debt
: Not all technical debt is the same. Intentional debt is a deliberate shortcut, taken to ship faster with a plan to revisit it later. Unintentional debt accumulates without anyone noticing: outdated dependencies, patterns that were never good, code that outgrew its original purpose.

  Knowing the difference changes how you respond. Intentional debt often has a ticket or comment explaining it. Unintentional debt usually doesn't. When you find debt with no explanation, don't assume carelessness. Find out whether it was a constraint you don't know about yet.

  Legacy codebases accumulate outdated technologies, deprecated patterns, and approaches that have since become anti-patterns. You'll need to work around them, refactor them, or accept them as constraints. Either way, have a plan to address them over time.

## Software archaeology

Software archaeology is the practice of reading and understanding an unfamiliar codebase. Before making changes, build a mental model of how the code works and why.

Code organization
: Start by understanding how the codebase is structured. Look for patterns in directory layout, naming conventions, and module boundaries. Organization reveals the author's intent.

Domain concepts
: Identify the business concepts expressed in the code. Class names, method names, and variable names often map directly to the problem domain. Understanding the domain helps you predict where to find things.

Tests
: Read the tests before the implementation. Tests describe what the code is supposed to do and how it behaves under specific conditions. They're often the most honest documentation in the codebase.

Callers
: To understand what a class or function does, look at how it's called. Callers reveal the intended contract: what goes in, what comes out, and what the caller expects in return.

  Use your IDE's **Find References** or **Find Usages** command to locate every call site. In Go, `gopls` provides a call hierarchy that shows both callers and callees directly in your editor.

Exceptions
: Be cautious with exception handling. Exceptions can obscure the normal flow of a program and mislead you about how the code actually behaves in practice.

  Watch for these patterns: catch blocks that do nothing (swallowed exceptions), overly broad catches that hide the specific failure, and error codes returned from functions that callers silently ignore. Each pattern means the code fails quietly, and you won't see it unless you look for it.

Most frequently modified code
: Start your investigation with the files that change most often. Frequent modification signals active business logic, which is usually where the most important behavior lives.

  Use this command to find the most frequently changed files in the repository:

  ```
  git log --name-only --pretty=format: | sort | uniq -c | sort -rn | head -20
  ```

  The output ranks files by commit frequency. Start at the top.

## Reading tests

Tests do more than verify correctness. They document the expected inputs and outputs of methods, the interactions between components, and the overall system behavior. Reading tests gives you a view of the codebase that source code alone can't provide.

Tests often map directly to user stories. When they do, they tell you what the system is supposed to accomplish from the user's perspective, not just how it's implemented.

Integration tests
: Integration tests show the big picture. They reveal how components work together and what the system expects at its boundaries. Look for tests that exercise an entire process from start to finish. They're often the clearest description of intended behavior.

Edge cases
: An edge case is a situation that occurs at the extreme boundary of a program's expected input, operating conditions, or usage patterns. Edge cases are where assumptions break down. When you find tests that cover them, pay close attention. They often encode hard-won knowledge about where the system fails.

## Contributing to open source to learn

Contributing to open source projects is one of the most effective ways to accelerate your growth as a software engineer. You read production-quality code written by experienced developers, work within real constraints, and get feedback on your contributions from the community.

**Find a project.** Search GitHub for repositories labeled `good first issue` or `help wanted`. These labels mark tasks that maintainers have identified as suitable for new contributors. Filter by language to find projects in your area of focus.

**Read CONTRIBUTING.md first.** Most projects document their workflow, coding standards, and PR requirements there. Skipping it is the most common mistake new contributors make.

**Apply software archaeology before you change anything.** Read the tests, trace the callers, and check which files change most often. The techniques in the [Software archaeology](#software-archaeology) section apply directly here.

**Start small.** Fix a typo in documentation, improve an error message, or reproduce a reported bug. Small contributions build familiarity with the codebase and the contribution workflow before you tackle larger changes.

**Learn from the review process.** When maintainers review your PR, read their feedback carefully. You get direct insight into how experienced engineers think about code quality, design, and conventions.

**Further reading**

- [Everything I really needed to know I learned from contributing to open source](https://manifestcorp.com/really-need-know-learned-contributing-open-source/) — Lessons learned from real open source contributions, covering what the experience teaches that formal education doesn't.
- [How to Be an Open Source Rock Star (Or at Least a Local Celebrity)](https://www.youtube.com/watch?v=kHZzQR9ntvc) — A practical talk on building credibility and making meaningful contributions in the open source community.
