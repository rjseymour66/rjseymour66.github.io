---
title: "APIs"
weight: 210
description: >
  Implementation details for application programming interfaces (APIs).
---

## Versioning

You can version your API one of two ways:
- Prefix all URLs with the version:
  ```
  /v1/healthcheck
  ```
- Use custom `Accept` and `Content-Type` headers.

## RESTful routing

Routes with identical patterns use different handlers based on the HTTP request method. For example, consider the following route:
```
/v1/movies/:id
```
> A URL that uses interpolated parameters such as `:id` are called _clean URLs_. These are clean in comparison to URLs that use query parameters.
>
> Go's http.ServeMux type does not support clean URLs (use httprouter).


You can use the same route with both a `GET` and `PUT` request method. The `GET` method retrieves a specific movie by `id`, while the `PUT` method partially updates a specific movie.

### Request methods

The following table briefly describes common REST methods:

| Method | Description |
|:-------|:------------|
| GET    | Retrieve information, does not change state of the application or resource |
| POST   | Modifies state, generally to create a new resource. Non-idempotent. |
| PUT    | Idempotent, modifies state, generally modifies or replaces existing resource. |
| PATCH  | Partial updates, either idempotent or non-idempotent. |
| DELETE | Delete a resource at the specified URL. |
