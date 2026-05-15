+++
title = 'Ui Design'
date = '2026-05-11T21:41:00-04:00'
weight = 60
draft = false
+++

Good UI design goes beyond aesthetics. It determines whether your product can be used by everyone, regardless of ability, language, or location, and whether it meets legal requirements in the markets you serve.

## Accessibility, internationalization, and localization

### a11y

*a11y* is a numeronym for *accessibility*: there are 11 characters between the 'a' and the 'y'. Accessibility means designing products that people with disabilities can use. That includes visual impairments (blindness, low vision, color blindness), auditory impairments, motor impairments (limited use of hands, reliance on keyboard or switch navigation), and cognitive impairments.

The technical standard for web accessibility is the [Web Content Accessibility Guidelines (WCAG)](https://www.w3.org/WAI/standards-guidelines/wcag/), published by the W3C. WCAG defines three conformance levels: A, AA, and AAA. Level AA is the threshold required by most legal frameworks and the target for most products.

### i18n

*i18n* is a numeronym for *internationalization*: 18 characters between the 'i' and the 'n'. Internationalization is the process of designing and building your application to be adaptable to different languages and cultural expectations. It involves separating text from code, supporting Unicode, accommodating right-to-left scripts, and designing layouts flexible enough to handle varying text lengths.

Internationalization is the groundwork. You do it once, during initial design and development, so that localization becomes straightforward for every market you enter. A product that wasn't built with internationalization in mind requires significant rework to adapt. Layouts break when text expands in translation, hardcoded strings can't be replaced, and date or currency formatting is embedded in logic rather than driven by locale.

### l10n

*l10n* is a numeronym for *localization*: 10 characters between the 'l' and the 'n'. Localization is the process of adapting your interface to the cultural and linguistic expectations of a specific target market. It includes translating text, formatting dates, times, numbers, and currencies according to local conventions, and adapting cultural references, imagery, and color associations that may carry different meaning in different regions.

Where internationalization asks "can this product work in another language?", localization asks "does this product feel right to a user in this specific place?" The two are inseparable: internationalization makes localization possible, and localization is where that investment pays off.

## Legal requirements

Accessibility is not only a design practice. In many jurisdictions it is a legal obligation.

### Americans with Disabilities Act

The [Americans with Disabilities Act (ADA)](https://www.ada.gov/) is a civil rights law enacted in 1990. Title III prohibits discrimination against people with disabilities in places of public accommodation. Courts have consistently held that websites and digital products fall within that definition, making ADA compliance a legal requirement for most US businesses. The ADA does not specify a technical standard, but WCAG 2.1 Level AA is widely accepted by courts and regulators as the benchmark.

### Section 508 of the Rehabilitation Act

[Section 508](https://www.section508.gov/) requires federal agencies and organizations that receive federal funding to make their electronic and information technology accessible to people with disabilities. It explicitly references WCAG 2.0 Level AA as the technical standard. If your product is used by or sold to a federal agency, Section 508 compliance is mandatory.

### European Accessibility Act

The [European Accessibility Act (EAA)](https://ec.europa.eu/social/main.jsp?catId=1202) is an EU directive that extends accessibility requirements to private companies, not just public bodies. It covers websites, mobile applications, e-commerce, banking, and other digital services offered in EU markets. The EAA references the EN 301 549 standard, which maps directly to WCAG 2.1 Level AA. Compliance was required by June 28, 2025.

## Usability

*Usability* measures how well users can interact with your interface to accomplish their goals. It breaks down into six dimensions.

### Learnability

How easy is it for new users to learn your application? A learnable interface lets users accomplish basic tasks the first time they encounter it, without training or documentation. Clear labels, consistent patterns, and familiar conventions reduce the learning curve. If users need a tutorial to perform a core task, the interface has failed them.

### Efficiency

How efficiently can experienced users accomplish tasks? Once users have learned the interface, it should get out of their way. Efficiency means minimizing the number of steps, clicks, and decisions required to complete common workflows. Keyboard shortcuts, sensible defaults, and well-placed controls all contribute. Measure efficiency by how long it takes a practiced user to complete representative tasks.

### Memorability

How well can infrequent users remember how to use your application after returning to it? An interface with strong memorability doesn't require users to re-learn it every time they come back. Consistent navigation, recognizable icons, and predictable behavior help users re-orient quickly after a period of absence.

### Discoverability

How easy is it for users to find and understand the features your system offers? A discoverable interface surfaces functionality at the right moment without overwhelming the user. Progressive disclosure (showing only what is relevant until more is needed) keeps the interface approachable while making the full feature set findable. If users don't know a feature exists, it might as well not.

### Error handling

How does your application handle and recover from errors? The *happy path* is the ideal flow where everything works as expected. The moment a user steps off it — entering invalid input, hitting a network failure, or reaching a state the application didn't anticipate — your error handling is what determines whether they recover or give up.

Every deviation from the happy path deserves a meaningful error message. "Something went wrong" is not an error message. It tells the user nothing about what happened, why it happened, or what to do next. A meaningful error message does three things: it explains what went wrong in plain language, tells the user why it happened if that information helps them, and suggests a concrete next step or alternative.

| Unhelpful | Helpful |
|---|---|
| Something went wrong. | Your session expired. Sign in again to continue. |
| Invalid input. | Enter a date after today's date. |
| Error 422. | That email address is already registered. Sign in or reset your password. |

Your application should make it easy for users to recover. Where possible, support undo and revert. A user who can reverse a mistake doesn't need to fear making one. That confidence keeps them moving forward rather than abandoning the task. For destructive or irreversible actions, ask for confirmation before proceeding rather than relying on recovery after the fact.

Good error messages also reduce support burden. When users understand what went wrong and how to fix it, they fix it themselves. When they don't, they file a support ticket or leave.

### User satisfaction

How do users feel after working with your application? Satisfaction encompasses the emotional response to the overall experience: whether the interface feels responsive, polished, and respectful of the user's time. Users who are satisfied return. Users who are frustrated look for alternatives. Satisfaction is harder to measure than efficiency, but surveys, session recordings, and support ticket volume all provide signal.

### Accessibility

Does the interface enable assistive technologies? An accessible interface works for users who rely on screen readers, keyboard navigation, switch access, or other assistive tools. Usability and accessibility are not separate concerns. An interface that is unusable for someone with a disability is simply unusable. Design for the full range of users from the start, not as a compliance exercise at the end.

## Users and secondary users

Understanding who uses your product, and how, is the foundation of good interface design. Alan Cooper's framework, established in *About Face: The Essentials of Interaction Design*, distinguishes between primary and secondary users.

### Primary users

A *primary user* is someone who interacts directly and frequently with your interface to accomplish their own goals. They are the main audience the product is designed for. Primary users have a direct relationship with the system: they provide input, make decisions within it, and act on what it returns.

#### Questions to identify primary users

- Who uses this product most often, and in what context?
- What goals are they trying to accomplish when they open it?
- What tasks do they perform repeatedly?
- What is their level of technical proficiency?
- What constraints do they work under: time pressure, environment, device, connectivity?
- What does failure look like for them? What happens when the product doesn't work?

### Secondary users

A *secondary user* interacts with the product less frequently, or interacts with the outputs of primary users rather than with the system directly. A manager who reviews reports generated by a data entry clerk is a secondary user. A customer who receives an invoice created through an accounting system is a secondary user. They are affected by the product's design without necessarily touching it themselves.

Secondary users are easy to overlook during design because they don't drive the product's primary workflows. Overlooking them leads to interfaces that work well for the primary user but produce outputs the secondary user can't understand or act on.

#### Questions to identify secondary users

- Who receives, reads, or acts on the output that primary users produce?
- Who manages or supervises primary users and needs visibility into their work?
- Who interacts with the product occasionally rather than as part of a regular workflow?
- What do secondary users need from the product that primary users don't?
- What happens to a secondary user when the primary user makes an error?

### Identifying users in practice

The Nielsen Norman Group recommends *user interviews* and *contextual inquiry* (observing users in their actual environment while they work) as the most reliable methods for identifying who your real users are. Secondary users often surface only through this kind of direct observation, because primary users frequently don't think to mention the people downstream of their work.

A practical starting point is to ask one question to every stakeholder you interview: *who else is affected by what you do in this system?* The answers reliably surface secondary users that no requirements document will name.

## Principles of design

The four principles of design — contrast, repetition, alignment, and proximity — were formalized by Robin Williams in *The Non-Designer's Design Book*. Together they account for most of the difference between interfaces that feel polished and those that feel accidental.

### Contrast

Contrast is one of the most effective tools available to you. Any two elements that differ from each other create contrast, but weak contrast is almost worse than none. If two things are different, make them *really* different. A heading that is only slightly larger than body text reads as inconsistent rather than intentional. A heading that is dramatically larger, heavier, or in a different color reads as hierarchy.

Use contrast to direct attention. The element with the most contrast draws the eye first. That should always be the most important element on the screen. Use contrast between type sizes to establish hierarchy, between colors to separate sections, and between weights to distinguish labels from values.

Contrast also serves accessibility. Text with insufficient contrast between foreground and background fails WCAG requirements and is genuinely hard to read for users with low vision.

### Repetition

Repetition provides a sense of familiarity and cohesiveness. When users encounter the same visual pattern repeatedly, they don't have to re-learn how to read the interface. That cognitive load is eliminated.

Start with the obvious: headings should share the same font, weight, and size at each level. Buttons of the same type should look identical. Icons should come from the same set. Then look further. Consistent spacing between sections, consistent padding inside cards, consistent use of color for interactive elements — each repetition reinforces the system and builds trust with the user.

Don't overdo it. Repetition creates consistency, not monotony. Vary elements intentionally when you need to signal that something is different. The contrast between the repeated pattern and the exception is what gives the exception its meaning.

### Alignment

Place every element with care. Never drop something onto a screen because there happens to be space. Every element should have a visual connection to something else on the page.

Aligning elements to a left or right edge creates a sharp vertical line that runs through the layout. That line is invisible, but the eye perceives it and interprets it as order and intention. Sharp edges produce a polished, professional result. A layout with many different starting points — some centered, some left-aligned, some indented arbitrarily — feels unsettled even when the viewer can't explain why.

Avoid centering as a default. Centered text works for short headlines and formal contexts, but a page full of centered elements has no strong edges and reads as unanchored. Left-align body text and interface elements by default. Reserve centering for deliberate, specific use.

### Proximity

Elements that are grouped together are perceived as related, even if they are not. This is one of the most powerful principles in visual design, and one of the easiest to misuse.

Group related elements intentionally: a label and its input, a heading and the content it introduces, an icon and its caption. Separate unrelated elements with whitespace. The amount of space between elements communicates the strength of their relationship. Items with no space between them are read as a single unit. Items separated by generous whitespace are read as independent.

Use whitespace actively, not as leftover space. Whitespace is not empty. It is a design element that creates breathing room, establishes hierarchy, and guides the eye through the interface.

## Make the right thing obvious

*Discoverability* is the key to learning an application. When users can see what is possible, understand how to do it, and find what they need without instruction, they learn faster and make fewer errors. Discoverability is not a feature you add. It is a quality baked into every interaction you design.

### Frameworks for discoverability

Don Norman's foundational work in *The Design of Everyday Things* introduced two concepts that underpin discoverability: *affordances* and *signifiers*. An affordance is what an interface element allows the user to do. A signifier is the cue that communicates where and how to perform the action. A button affords clicking. Its raised appearance, label, and cursor change are its signifiers. When signifiers are absent or misleading, users have to guess.

Jakob Nielsen's tenth usability heuristic, *recognition over recall*, reinforces this directly. Users should not have to remember information from one part of the interface to use another. Make options visible. Surface choices rather than requiring users to remember commands, paths, or syntax.

*Progressive disclosure* complements both. Show users what they need for the current task. Reveal advanced options only when the context calls for them. A simpler initial view reduces the cognitive load of first use while keeping the full feature set accessible.

### Discoverability contributes to memorability

An interface that makes the right thing obvious is also easier to remember. When the correct action is clearly signaled, users build accurate mental models on first contact. Those models persist. When they return after an absence, the signifiers they encountered before re-activate the memory. An interface that forced guessing on first use forces guessing on every return.

### Use an obvious approach

When you have a choice between a clever solution and an obvious one, choose the obvious one. Familiar patterns lower the cost of learning to zero for users who have encountered them before. Novel patterns require investment every time.

Apply this practically:

- Use placeholder text to show the expected format before the user types
- Use field masks to guide input as it is entered

```
Phone:  (___) ___-____
Date:   MM/DD/YYYY
```

- Provide formatted examples adjacent to inputs where the expected value is not self-evident

```
API key   e.g. sk-a1b2c3d4e5f6
Subdomain e.g. mycompany.example.com
```

- Use visual cues — icons, color, shape, and position — to communicate meaning without relying on text alone
- Show inline validation as the user types rather than only after submission, so errors surface at the moment they can be corrected most easily

Every hint, mask, and example you provide is a question you've answered before the user had to ask it.

## Destructive actions

A destructive action is any action that removes, overwrites, or permanently alters data in a way that cannot be easily undone. Deleting a record, canceling an account, or clearing a dataset all qualify. These actions demand more friction than ordinary interactions, not less.

Do not make it easy for users to destroy important data. A delete button that triggers immediate deletion with a single click is a design failure. The interface should require users to confirm that they understand what they are about to do and that they actually intend to do it.

### Add steps to create certainty

Use multiple steps to slow the action down and give the user space to reconsider:

1. **Initiate.** The user clicks a delete or remove action. The button should be visually distinct — typically a warning color — so it doesn't blend in with standard controls.
2. **Confirm.** Present a confirmation dialog that clearly states what will be deleted and that the action cannot be undone. Do not use vague language like "Are you sure?" Name the specific item: "Delete project Acme Q1 Report? This cannot be undone."
3. **Verify intent for high-stakes actions.** For irreversible or high-impact deletions, require the user to type the name of the resource or the word "delete" before the action proceeds. This extra step eliminates accidental confirmation.

```
To delete this account, type "delete my account" below:

[ _________________________ ]

[ Cancel ]  [ Delete account ]
```

### Design the controls to match the stakes

Place destructive actions away from safe actions. A delete button adjacent to a save button invites mistakes. Put distance — spatial or visual — between them. Use color to signal danger, but never rely on color alone. Pair it with an icon and a clear label.

Disable the final confirmation button until any required verification input is complete. An enabled button implies the action is ready to proceed. If the user hasn't met the confirmation requirement, the button should not suggest otherwise.
