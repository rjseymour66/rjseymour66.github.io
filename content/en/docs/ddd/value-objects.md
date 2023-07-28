---
title: "Value objects"
linkTitle: "Value objects"
weight: 3
description: >
  How to use value objects
---

Value objects are not unique. When deciding whether to model something as a value object, ask the following questions:
- Is this object immutable?
- Does it measure, quantify, or describe a domain concept?
- Can it be compared to other objects of the same type by just its values?

If you are still unsure, you can treat something as a value object and then upgrade it to an entity later.

```go
// Product is a value object that models a CoffeeCo product.
type Product struct {
	ItemName  string
	BasePrice money.Money
}
```