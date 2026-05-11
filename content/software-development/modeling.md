+++
title = 'Modeling'
date = '2026-05-10T20:22:50-04:00'
weight = 30
draft = false
+++

Throughout a project, you use models and box-and-line diagrams to express technical intent. Your challenge is knowing what diagram to create and when to create it.

## What is software modeling?

*Software modeling* is the process of creating abstract representations of a software system to better understand, analyze, and communicate its structure, behavior, and functionality.

Jack Reeves argued in his 1992 essay "What Is Software Design?" that programming is fundamentally a design activity. Writing code is not manufacturing. It is design. The purest expression of that design is the source code itself. Everything else, including diagrams, is scaffolding that supports the design process.

That framing clarifies what models are for. A diagram is not the deliverable. It is a thinking tool. You reach for it when the problem is too complex to hold in your head, when you need to communicate intent to a team before writing code, or when you need to reason through a decision before committing to it.

Diagrams serve several purposes throughout a project:

- **Context.** They place a system in relation to the people and systems around it, making scope and boundaries visible.
- **Complexity management.** They let you decompose a large problem into smaller, understandable parts without losing sight of how those parts connect.
- **Problem decomposition.** They help you identify the right boundaries between components before those boundaries are encoded in the codebase and become expensive to change.
- **Quality attribute prediction.** They let you reason about non-functional requirements such as performance, scalability, availability, and security before the system is built. A diagram that reveals a synchronous chain of six service calls predicts a latency problem. A diagram that shows a single database shared by five services predicts a scaling constraint.

The challenge is not learning to draw diagrams. It is knowing which diagram serves the current decision, and when the decision is better served by writing code instead.

## Types of diagrams

### Context diagrams

A *context diagram* places your system in relation to the people and external systems around it. Your system appears as a single box. Everything else — users, third-party services, other internal systems — surrounds it, connected by labeled arrows that describe the interactions.

Use a context diagram at the start of a project to establish scope and surface assumptions. It answers the question every stakeholder needs answered first: what does this system do, and what does it touch? It is also useful when onboarding a new team member who needs the big picture before any detail.

### Component diagrams

A *component diagram* zooms into a single service or application and shows its internal structure: the controllers, services, repositories, and other building blocks that make it work, and the dependencies between them.

Use a component diagram when designing a significant change to a single service, when planning a refactor, or when explaining how responsibilities are divided to a developer joining the team. It bridges the gap between the high-level architecture and the code.

### Class diagrams

A *class diagram* shows the classes in your codebase, their attributes and methods, and the relationships between them: inheritance, composition, and association.

Use a class diagram when designing a domain model before writing code, or when the relationships between types are complex enough that a diagram communicates them faster than reading the source. They are most valuable as a design tool early in a project, when the cost of changing the model is still low.

### Sequence diagrams

A *sequence diagram* shows how components interact over time. Each participant appears as a vertical line. Messages between them are horizontal arrows, ordered from top to bottom, showing what calls what and in what sequence.

Use a sequence diagram when order matters: designing a new API interaction, explaining an authentication flow, or diagnosing a race condition. Draw one scenario per diagram. A sequence diagram that attempts to show all possible paths becomes unreadable. Start with the happy path, then draw a separate diagram for the most critical error case.

### Deployment diagrams

A *deployment diagram* shows where your software runs: the servers, containers, cloud services, load balancers, and networks that make up your production environment, and how traffic flows between them.

Use a deployment diagram when planning infrastructure for a new system, documenting an existing production environment for operational reference, or reasoning about availability and failure modes. A deployment diagram makes it immediately visible when a single point of failure exists or when a network boundary introduces latency.

### Data models

A *data model* (often expressed as an entity-relationship diagram) shows the entities in your domain, the attributes they carry, and the relationships between them.

Use a data model before designing a database schema to validate that the model reflects the domain correctly. Getting the model right at this stage prevents expensive migrations later. Data models are also useful for communicating the structure of a system to developers and stakeholders who need to understand the data without reading the schema directly.

## Tools

### [OmniGraffle](https://www.omnigroup.com/omnigraffle)

A Mac and iPad diagramming tool for precise, polished diagrams. Use OmniGraffle when you need professional-quality output: architecture overviews, component diagrams, or any diagram you're presenting to stakeholders. It offers fine-grained control over layout and styling that web-based tools don't match.

### [Visio](https://www.microsoft.com/en-us/microsoft-365/visio/flowchart-software)

Microsoft's diagramming tool and the enterprise standard in many organizations. Use Visio for network diagrams, infrastructure layouts, and process flows, especially when your organization is Microsoft-heavy or when you need to share diagrams with stakeholders who expect Visio format.

### [Mural](https://www.mural.co)

A collaborative online whiteboard built for workshops and distributed teams. Use Mural in the early stages of a project for architecture brainstorming, design workshops, and team alignment sessions. It prioritizes collaboration over precision, making it well-suited for exploration rather than documentation.

### [Miro](https://miro.com)

Similar to Mural with stronger diagramming support. Use Miro for remote workshops, user journey mapping, and lightweight architecture discussions. It integrates well with tools like Jira and Confluence, making it a natural fit for teams already in the Atlassian ecosystem.

### [Lucidchart](https://www.lucidchart.com)

A web-based diagramming tool that covers the full range of diagram types. Use Lucidchart when you need a versatile, shareable tool that integrates directly with Google Workspace or Confluence. It balances ease of use with enough precision for technical documentation.

### [Mermaid](https://mermaid.js.org)

A text-based diagramming tool that renders diagrams from code embedded in Markdown. Use Mermaid for sequence diagrams and flowcharts that live alongside your documentation in a repository. Because the diagram is text, it is version-controlled, diff-able, and reviewable in a pull request like any other source file.

### [Structurizr](https://structurizr.com)

A purpose-built tool for C4 model diagrams, created by Simon Brown, the author of the C4 model. Use Structurizr when you are diagramming architecture systematically across context, container, and component levels. Its DSL lets you define your architecture once and render multiple diagram views from a single model.

### [PlantUML](https://plantuml.com)

A text-based tool for generating UML diagrams from plain-text descriptions. Use PlantUML for sequence diagrams, class diagrams, and state diagrams embedded in documentation. It integrates with Confluence, IntelliJ, VS Code, and most documentation pipelines, making it a strong choice when diagrams need to live close to the code.
