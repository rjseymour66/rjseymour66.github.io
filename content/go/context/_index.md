+++
title = 'Context'
date = '2026-02-25T23:07:21-05:00'
weight = 80
draft = false
+++

- Need to propogate cancellation signals to stop ongoing operations when they're no longer needed, such as when users cancel requests of there is a timeout.
- 


## Basics

`Context` is an idiomatic way to propagate cancellation signals through an app. It is like a tree, with a root context, branches, and leaves. A cancellation propagates through the hierarchy. If you cancel a parent, all children contexts are cancelled. However, if you cancel a child context, the parent context is untouched.


A root `Context` is immutable---you can't cancel it---but you can derive a cancellable `Context` from it. For example:
1. Gets a root context.
2. Creates a cancellable context from `root`. `WithTimeout` takes a parent context and a `time.Duration` timeout. This context cancels automatically after 15 seconds.
   
   ```go
   root := context.Background()                                         // 1
   child, cancel := context.WithTimeout(root, 15*time.Second)           // 2
   grandChild, cancel := context.WithTimeout(root, 5*time.Second)       // 3
   ```

### Create a context

This table summarizes the ways to create a context:

| Function                              | Creates             | Usage                                      | Auto-Cancels?        | Cancel Func? |
| :------------------------------------ | :------------------ | :----------------------------------------- | -------------------- | :----------- |
| `context.Background()`                | Root context        | In main, app startup, tests                | No                   | No           |
| `context.TODO()`                      | Placeholder root    | When refactoring or undecided              | No                   | No           |
| `context.WithCancel(parent)`          | Cancelable child    | Manual cancellation (goroutines, shutdown) | No (manual only)     | Yes          |
| `context.WithTimeout(parent, d)`      | Child with timeout  | Limit operation duration                   | Yes (after duration) | Yes          |
| `context.WithDeadline(parent, t)`     | Child with deadline | Cancel at specific time                    | Yes (at deadline)    | Yes          |
| `context.WithValue(parent, key, val)` | Child with value    | Request-scoped metadata                    | No                   | No           |


### Context methods

| Method           | Returns           | Description                     | Usage                       |
| :--------------- | :---------------- | :------------------------------ | :-------------------------- |
| `ctx.Done()`     | <-chan struct{}   | Closes when canceled or expired | In select to stop work      |
| `ctx.Err()`      | error             | Why it was canceled             | After Done() fires          |
| `ctx.Deadline()` | (time.Time, bool) | Reports deadline if set         | Rare — mostly internal libs |
| `ctx.Value(key)` | any               | Gets stored value               | Request metadata            |

## Context in practice

- Passes as the first parameter for consistency