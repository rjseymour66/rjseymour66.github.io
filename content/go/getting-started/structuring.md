+++
title = 'Structuring packages and services'
date = '2026-03-03T21:31:48-05:00'
weight = 20
draft = false
+++

For a thorough reference, see [Organizing a Go module](https://go.dev/doc/modules/layout).

A package is a unit of responsibility with a single, clear purpose. Design packages around what they *do*, not what they *are*. A package named `utils` or `helpers` is a warning sign. It has no defined purpose and will accumulate unrelated code over time.

This page covers how to design, name, and divide packages. For project layout and dependency direction, see [Project setup](../project-setup).

## internal/

The `internal/` directory restricts which packages can import yours. Only code within the parent of `internal/` can import packages inside it. The Go compiler enforces this. Any violation is a compile error.

```bash
myapp/
├── internal/
│   ├── domain/    # importable by anything under myapp/
│   └── store/     # importable by anything under myapp/
└── cmd/
    └── myapp/     # can import internal/domain and internal/store
```

A separate module at the same level as `myapp/` cannot import anything under `myapp/internal/`. This makes it safe to change internal packages without worrying about breaking external consumers.

Put packages in `internal/` when they:
- Implement details that shouldn't be part of your public API.
- Depend on infrastructure (databases, HTTP) that callers shouldn't need to know about.
- Are not stable enough to commit to as a public interface.

Leave packages outside `internal/` only when you intend other modules to import them directly: for example, a shared client library or a public SDK.

## Package naming

Package names are lowercase, single words with no underscores or mixed caps. The name is part of every identifier exported from the package. Choose a name that reads naturally at the call site.

```go
// good: the name reads clearly at the call site
rest.NewHandler(...)
store.NewUserStore(...)
domain.ErrNotFound

// avoid: redundant names that stutter
userstore.NewUserStore(...)   // "userstore.UserStore" repeats "user"
resthandler.New(...)          // "handler" is already implied by "rest"
```

Follow these rules from [Effective Go](https://go.dev/doc/effective_go#package-names):

- **Name the package for what it provides, not what it contains.** `encoding/json` encodes and decodes JSON. It's not named `jsonutils` or `jsonhelpers`.
- **Avoid generic names.** `util`, `common`, `misc`, `base`, and `shared` describe nothing. If you can't name the package, the package probably shouldn't exist yet.
- **Avoid repeating the package name in identifiers.** If your package is `store`, name the constructor `store.New`, not `store.NewStore`.
- **Prefer nouns for packages that provide types**, verbs or noun phrases for packages that provide operations: `http`, `io`, `sort`, `sync`.

## Package size

A package is too large when:
- It has more than one reason to change. If adding a new feature requires editing the package for unrelated reasons, split it.
- It imports packages it uses in only one file. Move that file and its imports to a new package.
- Readers need to understand the whole package to use any part of it.

A package is too small when:
- It exports only one or two identifiers used by a single other package. Merge it upward.
- Its name only makes sense in the context of another package (`uservalidation` alongside `user`). Merge the validation into the `user` package.

The right size is a package that a new reader can understand in full after reading its exported identifiers and one or two files.

## Common package patterns

These patterns appear in most production Go services. Use them as a starting point, not a prescription.

| Package   | Purpose                                 | What belongs here                                        |
| --------- | --------------------------------------- | -------------------------------------------------------- |
| `domain`  | Core types, validation, sentinel errors | `User`, `Order`, `Validate`, `ErrNotFound`               |
| `service` | Business operations                     | `UserService.Register`, `OrderService.Cancel`            |
| `store`   | Storage adapter                         | SQL queries, connection management, transaction handling |
| `rest`    | HTTP transport                          | Handlers, middleware, request/response types             |
| `kit`     | Shared utilities                        | Trace IDs, request logging, auth helpers                 |

- `domain` has no dependencies outside the standard library. Everything else depends on it. Never import `rest`, `store`, or `service` from `domain`.
- `service` depends on `domain` and defines interfaces for its dependencies (`UserStore`, `Mailer`). It never imports `store` or `rest` directly.
- `store` depends on `domain`. It implements the interfaces that `service` defines, but it never imports `service`.
- `rest` depends on `domain` and defines interfaces for the service methods it calls. It never imports `store` or `service` directly.
- `kit` depends only on the standard library. Any package can import it.

## Dependency rules

Organize packages in layers. Higher layers depend on lower ones. Lower layers never depend on higher ones.

```bash
rest      →  service (interface)  →  domain
store     →  domain
kit       →  (standard library only)
cmd/app   →  all layers
```

### Resolving an import cycle

If two packages import each other, you have a cycle. Go refuses to compile it. Cycles usually mean one of three things:

- The dependency direction is wrong. If `store` imports `rest`, the dependency is inverted. Move the shared type to `domain` so both can import it without importing each other.
- The packages are too tightly coupled. Extract the shared behavior into a third package that both can import.
- The packages should be merged. If two packages always change together and always import each other, they're one package pretending to be two.

The most common fix is to move the shared type or interface to `domain` and have both packages depend on it independently.
