---
title: "Asynchronous JS"
linkTitle: "Async"
weight: 80
description:
---

An asynchronous program must stop computing while it waits for data to arrive, or for some other event to occur:
- For example, JS in the browser is event-driven--it waits for the user to do something and then runs code
- Historically, these events were handled with callbacks

There are three important JS features that help with async programs:
- Promises: objects that represent the result of an async operation that has not yet completed
- async/await keywords: let you structure your Promise-based code as if it is synchronous
- async iterators and the `for/await` loop: work with streams of async events with loops, looks synchronous

## Callbacks

