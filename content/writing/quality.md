+++
title = 'Quality docs'
date = '2025-11-22T09:07:54-05:00'
weight = 10
draft = false
+++


## Drafts

The purpose of a rough draft is to figure out which parts you understand well enough to document, and which parts you need to research.

## Titles

- Tasks: Use the verbal form: _Installing_, _configuring_,...
- Concepts: Use concrete words that summarize the concept.
- Referece: Noun or noun phrase that indicates the name of the item and a common noun.
- Troubleshooting: State the problem that the topic addresses. For example, _Cannot connect to the server_.

## Tasks

### Focus on user goals

{{< admonition "User-oriented tasks" tip >}}
A user-oriented task is a task that users want to perform whether or not they are using your product. For example, "Trimming strings" rather than "Using the `string` package to trim strings".

A task that starts with "Using" or "Working" is a function-oriented task.
{{< /admonition >}}

A _task_ is an activity that users do, and a _goal_ is an outcome that uses want. To make sure you are focusing on goals, find scenarios. A _scenario_ is a story that shows how a product contributes to solving a business problem. You define key goals for a product, then create a scenario (or path) that helps a user accomplish the goal.

To identify the tasks that are associated with a goal, start with a high-level task and then get more granular:
1. Identify your users.
2. Evaluate their goals.
3. Determine the tasks that those users need to do with your product to meet their goals. 
4. Evaluate to make sure that users have the content to achieve their goals regardless of how or with what product they do so.

### Introductions

Users need to understand why you are giving them task information. Give practical reasons.


### Steps

#### Substeps

When a step has substeps, the top-level step should indicate the purpose of the step. Use an action verb:
1. For each application, _configure_ the client to send data to the server:
   1. Install the...
   2. Open port number...

#### Optional or conditional steps

The best place to tell users that a step is optional is at the beginning. Conditional steps usually begin with "If":
1. (Optional) Set default values...


## Concepts

Concept topics provide the necessary vocabulary and context for what users are doing.

## Troubleshooting

Troubleshooting documentation must include the following sections:
- Symptoms of the issue
- Potential causes of the issue
- How to resolve the error situation

### Structure

Introduction
: Define the concept and explain its value to the user.




## Revising

You acheive clarity when you revise. Rewriting is when you replace, add, or delete parts of your draft.

### Long gerund phrases

Do not begin your sentence with a long gerund phrase:

#### Original

Importing data with a CSV file from the database is required if you do not connect to the API.


#### Revision

It takes too long to get to the empty verb _is_. Instead replace with a straighforward subject and verb. Here, the subject and verb appear after an introductory phrase that indicates a condition:

If you do not connect to the API, [you must import] data...

### Long coordinated phrases and clauses

Writing becomes unclear when you string together too many sentences with coordinating conjunctions. Break them up into smaller sentences.

An indication that you need to break up sentences is when you join too many ideas with _or_.


### Too many modifying phrases or clauses

If you have too many relative clauses (`who`, `which`, or `that`), create multiple sentences from the original:

#### Original

Use the grep command that searches text files recursively to find the string.

#### Revision

Use the grep command to find the string. It can search text files recursively.


### Convert prepositional phrases to clauses

Prepositional phrases can modify a sentence too much. It can make the sentence difficult to comprehend. The following example converts a prepositional phrase to a clause.

#### Original

To handle long response times (during periods) (of heavy traffic) (on our APIs), the developers looked through the source code (in the company's repository).

#### Revision

To handle long response times _when API traffic was heavy_, the developers looked through the source code.


### There is/are

This construction is called an _expletive_. It is usually paired with a relative clause that uses "are" as well: "There are ... that are...".

To revise, just remove the expletive and relative pronoun.

#### Original

There are several types of servers that are applicable to your environment.

#### Revise

Several types of servers are applicable to your environment.


### Simpler words

Many complicated words are derived from Latin. Replace latin words with words or phrasal verbs derived from Anglo-Saxon:

| Latin-derived       | Anglo-Saxon replacement   |
| ------------------- | ------------------------- |
| accelerate          | speed up                  |
| accomplish          | do                        |
| acquire             | get                       |
| align               | line up                   |
| anticipate          | expect                    |
| approximate         | about, roughly            |
| attempt             | try                       |
| complete            | finish, fill in, fill out |
| configure           | set up                    |
| consider            | think about               |
| construct,fabricate | make                      |
| defer               | put off                   |
| demonstrate         | show                      |
| disconnect          | cut off                   |
| discover            | find, realize             |
| distribute          | give out, send            |
| documentation       | docs, help text           |
| enumerate           | list                      |
| exclude             | leave out                 |
| evident             | clear, plain              |
| functionality       | features, ability         |
| generate            | make                      |
| implement           | do, build, put in place   |
| indicate            | show, point out           |
| initiate, commence  | start, begin              |
| investigate         | look into                 |
| locate              | find                      |
| maintain            | keep, upkeep              |
| modify              | change                    |
| notify              | tell                      |
| optimize            | make better, tune         |
| perform             | do                        |
| possess             | have                      |
| provide             | give                      |
| present             | give, show, tell          |
| require             | need                      |
| retain              | keep                      |
| review              | look at                   |
| specify             | state, set                |
| sufficient          | enough                    |
| terminate           | end, stop                 |
| transmit            | send                      |
| utilize             | use                       |
| validate            | check, make sure          |
| verify              | check                     |


### "that" in relative clauses

Always use "that" in a relative clause. It helps with translation and helps clarify how parts of the sentence relate to each other.

#### Original

Each error message explains the issue it addresses.

#### Revision

Each error message explains the issue _that_ it addresses.

### Parentheses

Periods go inside parentheses when what is inside the parentheses is a complete sentence.

### Lists

Follow these steps:
- Use a complete sentence for the lead-in sentence.
- Start each list item with a capital letter unless the first word is never capitalized.
- Do not nest more than two levels