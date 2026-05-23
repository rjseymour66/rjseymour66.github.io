+++
title = 'Architecture'
date = '2026-05-22T22:00:45-04:00'
weight = 90
draft = false
+++

The architect is responsible for the overall design of the system, accounting for requirements that often go unstated: scale, performance, security, and other quality attributes sometimes called the *ilities*. Architecture is about making decisions that are difficult to change later, based on careful analysis of trade-offs.

Before designing a system, ask these questions to understand its real requirements:

- How many users will this system support? How many concurrently?
- Where are users located? Does the system need geographic distribution?
- What is the availability target? How much downtime is acceptable per year?
- What can cause traffic spikes? Scheduled events, marketing campaigns, or viral growth?
- When will this application be used? Business hours only, or around the clock?
- What regulatory or compliance constraints apply?
- What is the expected data volume now and in three years?
- What are the latency requirements? Is a 500ms response acceptable, or must it be under 50ms?
- What integrations exist with external systems, and what are their reliability guarantees?
- What is the team's experience with the proposed technology stack?

## Trade-offs

There are no right or wrong answers in architecture, only trade-offs. Choosing strong consistency means accepting reduced availability. Choosing a microservices architecture means accepting operational complexity in exchange for independent deployability. Choosing a managed cloud service means accepting vendor dependency in exchange for reduced operational burden.

When someone asks which database, which architecture, or which pattern to use, the honest answer is usually "it depends." What it depends on are the specific requirements, constraints, and priorities of the system you are building.

## Architecture vs design

Architecture and design are not separate activities. Design decisions affect the architecture, and architectural constraints shape design decisions. This relationship is sometimes called the *twin peaks model*: requirements and architecture evolve together, each informing the other.

Thinking as an architect means balancing two modes:

- *Strategic thinking* focuses on long-term goals and the overall direction of the project. It considers how the system will need to evolve, what decisions are hard to reverse, and which quality attributes are non-negotiable.
- *Tactical thinking* addresses immediate, acute problems. Tactical decisions are often necessary under time pressure but may accumulate as technical debt that requires a longer-term architectural solution later.

The tension between strategic and tactical is permanent. Recognize when you are making a tactical decision and record the intent to revisit it.

## Quality attributes

Quality attributes define how well a system performs its functions beyond basic correctness. They are sometimes called non-functional requirements, architectural characteristics, quality goals, constraints, quality of service goals, architecturally significant requirements, or the *ilities*.

Common quality attributes include:

- *Scalability*: the ability to handle increased load by adding resources
- *Availability*: the proportion of time the system is operational and accessible
- *Reliability*: the ability to perform correctly over time, including failure recovery
- *Performance*: response time, throughput, and resource utilization under expected load
- *Security*: protection against unauthorized access, data breaches, and malicious use
- *Maintainability*: how easily the system can be modified, debugged, and extended
- *Testability*: how easily the system's behavior can be verified
- *Deployability*: how quickly and safely changes can be released to production
- *Observability*: how well the internal state of the system can be understood from its outputs
- *Portability*: the ease of moving the system between environments or platforms

Quality attributes often conflict. Maximizing security can reduce performance. Maximizing scalability can reduce consistency. The architect's job is to identify which attributes matter most for the specific system and make trade-offs accordingly.

## Architectural styles

An *architectural style* is a set of design principles that defines how a system is organized and how its components interact. Common styles include layered (n-tier), microservices, event-driven, service-oriented, and monolithic architectures. Each style trades certain qualities for others: microservices trade operational simplicity for independent scalability and deployability, while a monolith trades deployment flexibility for simplicity.

Hypothesis-driven development applies the scientific method to architectural choices. Rather than committing to a style based on assumptions, state your hypothesis explicitly and define how you will measure success:

> We believe [architectural decision]
> will result in [expected outcome].
> We will know we succeeded when we see [specific metric] change.

For example:

> We believe decomposing the order service into a separate microservice
> will result in faster deployment cycles for the orders team.
> We will know we succeeded when order service deploys drop from weekly to daily.

Framing decisions as hypotheses forces you to define success before committing to an approach and gives you criteria to evaluate whether the decision delivered its intended value.

## Fitness functions

A *fitness function* is an automated test that measures how closely your architecture meets its objectives. The term comes from evolutionary computing, where a fitness function scores how well a candidate solution satisfies the problem constraints.

In software architecture, fitness functions verify that a refactoring or new feature does not violate architectural rules. They run continuously in your CI pipeline and alert you when something drifts out of bounds. Examples include:

- verifying that no package in the service layer imports the database layer directly
- checking that response times stay under a defined threshold under load
- ensuring that no circular dependencies are introduced between packages

In Go, fitness functions are ordinary tests that use the `testing` package and run with `go test`. What makes them architectural is what they assert: structure, dependencies, and constraints rather than business logic.

This example verifies that the service layer never imports the database layer directly:

```go
func TestServiceLayerDoesNotImportDB(t *testing.T) {
    pkgs, err := packages.Load(&packages.Config{
        Mode: packages.NeedImports,
    }, "./internal/service/...")
    if err != nil {
        t.Fatal(err)
    }
    for _, pkg := range pkgs {
        for imp := range pkg.Imports {
            if strings.Contains(imp, "internal/db") {
                t.Errorf("service package %s imports db layer: %s", pkg.PkgPath, imp)
            }
        }
    }
}
```

This example checks that no package exceeds a maximum allowed number of dependencies, catching packages that are accumulating too many responsibilities:

```go
func TestMaxDependenciesPerPackage(t *testing.T) {
    const maxDeps = 10
    pkgs, err := packages.Load(&packages.Config{
        Mode: packages.NeedImports,
    }, "./internal/...")
    if err != nil {
        t.Fatal(err)
    }
    for _, pkg := range pkgs {
        if count := len(pkg.Imports); count > maxDeps {
            t.Errorf("package %s has %d dependencies (max %d)", pkg.PkgPath, count, maxDeps)
        }
    }
}
```

Both examples use `golang.org/x/tools/go/packages` to inspect the module graph at test time. Add these tests to your CI pipeline so architectural drift is caught automatically rather than discovered during code review.

## Architectural decision records

An *architectural decision record* (ADR) documents a significant architectural decision and the reasoning behind it. The why matters more than the how: code shows what was built, but an ADR explains why a particular path was chosen over the alternatives considered.

A basic ADR contains:

- *Title*: a short, descriptive name for the decision
- *Status*: proposed, accepted, deprecated, or superseded
- *Context*: the forces and constraints that made this decision necessary
- *Options*: the alternatives considered and their trade-offs
- *Decision*: the choice made and the rationale
- *Consequences*: what becomes easier and harder as a result
- *Governance*: who owns this decision and who must be consulted before it changes
- *Notes*: links to related ADRs, relevant documentation, or discussion threads

Example:

> **ADR-001: Use PostgreSQL as the primary database**
>
> **Status:** Accepted
>
> **Context:** The system needs a relational database to support transactional operations and complex queries across related entities. The team has existing PostgreSQL expertise. The system will initially run on a single cloud provider.
>
> **Options:**
> - *PostgreSQL*: mature, full-featured, strong consistency, excellent tooling. Requires self-management or a managed service.
> - *MySQL*: similar capabilities, wider hosting availability, less advanced feature set.
> - *DynamoDB*: fully managed and horizontally scalable, but its key-value model does not fit the relational data requirements.
>
> **Decision:** Use PostgreSQL. The team's existing expertise reduces onboarding time, and the relational model matches the domain.
>
> **Consequences:** Future migrations away from PostgreSQL will require significant effort. The team must manage schema migrations carefully. Connection pooling using PgBouncer will be required at scale.
>
> **Governance:** The platform team owns this decision. Changes require review by the engineering lead.
>
> **Notes:** See ADR-004 for the connection pooling decision.

*Architectural katas* are structured practice exercises that simulate real architectural decision-making. A kata presents a fictional system brief with requirements, constraints, and user stories. Participants work in small groups to design an architecture, then present and defend their decisions. They are an effective way to build architectural thinking skills in a low-stakes environment. Neal Ford and Mark Richards maintain a library of katas at [architecturalkatas.com](https://architecturalkatas.com).
