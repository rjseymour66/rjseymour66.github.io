---
title: "Project structure"
weight: 5
description: >
  How to plan and organize JavaScript projects, from single-file scripts to full applications.
---

Before writing any code, decide what you are building. JavaScript projects fall into two
categories. A *script* is a focused, self-contained file that performs a single task. It requires
no build step, minimal dependencies, and no module system. An *application* is a collection of
modules, assets, and configuration that solves a broader problem. Applications benefit from a
build step, package management, and deliberate directory organization. The choice affects every
structural decision that follows.

## Scripts vs. applications

### When to write a script

Write a script when the project fits in a single file (typically under 200 lines), you need no
external dependencies, and no build output is required. Good candidates include automation
utilities, data transformation pipelines, and quick experiments.

A script that reads a CSV file, transforms each row, and writes the output to a new file is a
typical example.

### When to build an application

Build an application when any of the following apply:

- The project has multiple source files or modules
- You import third-party packages from npm
- The project targets the browser and requires bundling
- The codebase will be maintained and tested over time

A task management web app, a REST API server, or a data dashboard are all applications.

---

## Structuring a script

A well-structured script separates setup from logic and logic from I/O. This separation makes
the script easy to test and adapt.

### Script anatomy

A script follows this order:

1. Imports and configuration constants at the top
2. Pure utility functions in the middle
3. I/O operations at the bottom, called from a single `main()` or `run()` function

The following script fetches open GitHub issues and writes a summary to a file. The structure
separates the fetch logic, the formatting logic, and the file write into distinct sections:

```js
import { promises as fs } from 'node:fs';

// config
const REPO = 'owner/repo';
const OUTPUT_FILE = 'issues-summary.txt';

// utility — pure, no side effects
function formatIssue(issue) {
    return `#${issue.number}: ${issue.title} (${issue.state})`;
}

// I/O — runs once, at the bottom
async function run() {
    const res = await fetch(`https://api.github.com/repos/${REPO}/issues`);
    if (!res.ok) throw new Error(`GitHub API error: ${res.status}`);

    const issues = await res.json();
    const lines = issues.map(formatIssue).join('\n');

    await fs.writeFile(OUTPUT_FILE, lines, 'utf8');
    console.log(`Wrote ${issues.length} issues to ${OUTPUT_FILE}`);
}

run().catch(console.error);
```

This pattern works for Node.js CLI scripts, automation scripts, and data transformation
utilities. The `formatIssue` function is pure—give it the same input and it always returns the
same output—which means you can test it without any network calls or file access.

---

## Application directory structure

There is no universal layout for JavaScript applications, but two patterns handle most cases.

### Layer-based structure

*Layer-based structure* organizes code by technical role. Each directory represents one layer of
the stack:

```
src/
├── api/           # HTTP requests and external service calls
├── components/    # Reusable UI elements (for browser apps)
├── services/      # Business logic
├── utils/         # Pure helper functions
├── config/        # App-wide settings and constants
└── index.js       # Entry point
```

Layer-based structure works well for small projects (under five developers, under 5,000 lines)
where developers regularly work across all layers.

### Feature-based structure

*Feature-based structure* organizes code by product capability. Each feature directory contains
everything that feature needs: its API calls, business logic, and UI components. This approach
scales better on larger teams because each feature is self-contained:

```
src/
├── features/
│   ├── auth/
│   │   ├── auth.api.js
│   │   ├── auth.service.js
│   │   └── auth.ui.js
│   ├── dashboard/
│   │   ├── dashboard.api.js
│   │   ├── dashboard.service.js
│   │   └── dashboard.ui.js
│   └── tasks/
│       ├── tasks.api.js
│       ├── tasks.service.js
│       └── tasks.ui.js
├── shared/        # Code shared across features
├── config/
└── index.js
```

Choose feature-based structure when different developers own different parts of the product, or
when the codebase is growing toward six-figure line counts.

---

## Module system

JavaScript has two module systems in active use: *ECMAScript Modules* (ESM) and *CommonJS* (CJS).

### ESM

ESM is the standard for the web and for Node.js 14+. Each ESM file has its own scope. Nothing is
global by default. You explicitly export what you want to share and import what you need:

```js
// math.js — exporting
export function add(a, b) { return a + b; }
export const PI = 3.14159;

// main.js — importing
import { add, PI } from './math.js';
console.log(add(2, 3)); // 5
```

To enable ESM in Node.js, either name your files with the `.mjs` extension or add
`"type": "module"` to `package.json`. For new projects, always prefer ESM.

#### Named exports vs. default exports

Named exports are preferable for most cases because they make imports explicit and searchable
across editors and tooling:

```js
// named — preferred
export function fetchUser(id) { /* ... */ }
export function deleteUser(id) { /* ... */ }

// default — useful for a module's single primary export
export default class UserService { /* ... */ }
```

Avoid mixing default and named exports in the same file. Mixing the two creates inconsistency
and makes auto-import tools less reliable.

#### Dynamic imports

*Dynamic imports* load a module at runtime rather than at parse time. This technique enables
code splitting and lazy loading of large features:

```js
// Load the charting library only when the user opens the reports view
async function loadReports() {
    const { renderChart } = await import('./charts.js');
    renderChart(document.querySelector('#chart'), data);
}
```

Dynamic imports return a `Promise`, so they work naturally inside `async` functions.

### CommonJS

CJS is the original Node.js module system. CJS files run as scripts and require synchronous
loading:

```js
// math.js — exporting
function add(a, b) { return a + b; }
module.exports = { add };

// main.js — importing
const { add } = require('./math');
```

You will encounter CJS in legacy codebases and in many older npm packages. CommonJS remains
necessary when a package you depend on does not yet publish an ESM version.

---

## Entry points and initialization

Every application has an entry point: the file the runtime loads first. The entry point should
do as little as possible. Its job is to wire up the application and hand off to feature code.

### Browser entry point

A browser entry point waits for the DOM to be ready, reads configuration, and boots the
application. The following entry point for a task manager app initializes authentication and
then renders the UI:

```js
// src/index.js

import { initAuth } from './features/auth/auth.service.js';
import { renderApp } from './features/tasks/tasks.ui.js';
import { reportError } from './shared/error-reporter.js';

// Read server-injected config (see the environment page for this pattern)
const config = JSON.parse(document.getElementById('app-config').textContent);

window.addEventListener('DOMContentLoaded', async () => {
    try {
        await initAuth(config.apiBase);
        renderApp(document.querySelector('#root'));
    } catch (err) {
        reportError(err);
    }
});
```

### Node.js entry point

A Node.js entry point reads environment variables, connects to external services, and starts the
HTTP server. The following example boots an Express API server and establishes the database
connection before accepting traffic:

```js
// src/index.js

import { createApp } from './app.js';
import { connectDatabase } from './services/database.js';
import { logger } from './shared/logger.js';

const PORT = process.env.PORT ?? 3000;

async function start() {
    await connectDatabase(process.env.DATABASE_URL);
    logger.info('Database connected');

    const app = createApp();
    app.listen(PORT, () => logger.info(`Server listening on port ${PORT}`));
}

start().catch((err) => {
    logger.error('Failed to start server:', err);
    process.exit(1);
});
```

Calling `process.exit(1)` on startup failure is important for container runtimes. If the server
cannot connect to the database, the process should exit immediately so the orchestrator can
restart it or surface the error.

---

## Configuration and environment variables

Applications need different settings in development, testing, and production. Configuration
should come from the environment, not from source code.

### Environment variables in Node.js

Store secrets, service URLs, and environment-specific values in a `.env` file at the project
root:

```
DATABASE_URL=postgres://localhost:5432/myapp
API_KEY=abc123
PORT=3000
```

Load those values with the `dotenv` package before any other imports:

```js
import 'dotenv/config';

// process.env.DATABASE_URL is now available
```

Add `.env` to `.gitignore` immediately. Never commit credentials to version control. Provide a
`.env.example` file with placeholder values so other developers know which variables to set.

### Config module

Centralizing environment variable access in a single `config/index.js` module gives you one
place to validate required variables and provide defaults. A missing required variable fails at
startup with a clear message rather than silently at the point of first use:

```js
// src/config/index.js

const required = ['DATABASE_URL', 'API_KEY'];

for (const key of required) {
    if (!process.env[key]) {
        throw new Error(`Missing required environment variable: ${key}`);
    }
}

export const config = {
    db: {
        url: process.env.DATABASE_URL,
    },
    api: {
        key: process.env.API_KEY,
        baseUrl: process.env.API_BASE_URL ?? 'https://api.example.com',
    },
    port: Number(process.env.PORT) || 3000,
};
```

### Environment variables in browser apps

Browser apps cannot read `process.env` at runtime. Build tools like Vite inject environment
variables at build time from a `.env` file. In Vite, only variables prefixed with `VITE_` are
exposed to the browser bundle:

```
VITE_API_BASE_URL=https://api.example.com
VITE_ANALYTICS_KEY=xyz789
```

Access them in your code with `import.meta.env`:

```js
const apiBase = import.meta.env.VITE_API_BASE_URL;
```

Do not store secrets in browser environment variables. Any value injected at build time is
visible in the compiled bundle.

---

## Choosing a build tool

Build tools compile, bundle, and optimize your code for production. The right choice depends on
whether you are targeting the browser or Node.js, and whether you are building an application or
a library.

The following table compares the most common tools:

| Tool | Best for | Key strength |
|---|---|---|
| Vite | Browser applications | Near-instant dev server, native ESM in development |
| Webpack | Enterprise browser applications | Rich ecosystem, fine-grained control |
| esbuild | Node.js apps and CLIs | Extremely fast builds, minimal configuration |
| Rollup | Libraries | Clean ESM output, excellent tree shaking |

### Vite for browser applications

Vite is the default choice for new browser applications. It serves native ES modules during
development without bundling, so the dev server starts in milliseconds. For production, Vite
runs Rollup under the hood to produce an optimized bundle.

To scaffold a new Vite project, run the following command and answer the prompts:

```shell
$ npm create vite@latest my-app
```

The generated project follows this structure:

```
my-app/
├── public/          # Static assets served as-is (favicon, robots.txt)
├── src/
│   ├── main.js      # Entry point
│   └── style.css
├── index.html       # Root HTML template (Vite's entry point)
├── package.json
└── vite.config.js
```

Note that `index.html` lives in the project root, not inside `src/`. Vite treats `index.html`
as the entry point and resolves `<script type="module">` tags from there.

### esbuild for Node.js applications

esbuild compiles TypeScript and modern JavaScript to Node-compatible output with minimal
configuration. It is significantly faster than Webpack or Babel for typical Node.js workloads.

To configure esbuild for a Node.js API server, install the package:

```shell
$ npm install --save-dev esbuild
```

Then add build scripts to `package.json`:

```json
{
  "scripts": {
    "build": "esbuild src/index.js --bundle --platform=node --outfile=dist/index.js",
    "start": "node dist/index.js",
    "dev": "node --watch src/index.js"
  }
}
```

Node.js 18+ includes `--watch`, which restarts the process when source files change. For most
Node.js development, this eliminates the need for tools like `nodemon`.

---

## Code quality tools

Automated code quality tools enforce style consistently and catch errors before they reach
production. Configure both before writing any application code.

### ESLint

*ESLint* analyzes your source files for errors, style violations, and potential bugs. Configure
it in `eslint.config.js` using the flat config format introduced in ESLint 9:

```js
// eslint.config.js

import js from '@eslint/js';

export default [
    js.configs.recommended,
    {
        rules: {
            'no-unused-vars': 'error',
            'no-console': 'warn',
            'prefer-const': 'error',
            'eqeqeq': ['error', 'always'],
        },
    },
];
```

Run ESLint across your source directory with the following command:

```shell
$ npx eslint src/
```

### Prettier

*Prettier* formats code automatically. Unlike ESLint, Prettier does not analyze logic. It only
reformats text. Configure it in `.prettierrc`:

```json
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 4,
  "trailingComma": "es5",
  "printWidth": 100
}
```

### Combining ESLint and Prettier

Install `eslint-config-prettier` to disable any ESLint formatting rules that conflict with
Prettier:

```shell
$ npm install --save-dev eslint-config-prettier
```

Then add it to `eslint.config.js` to prevent the two tools from fighting each other:

```js
import js from '@eslint/js';
import prettierConfig from 'eslint-config-prettier';

export default [
    js.configs.recommended,
    prettierConfig,
    {
        rules: {
            'no-unused-vars': 'error',
            'prefer-const': 'error',
        },
    },
];
```

Add convenience scripts to `package.json` so you can run both tools from the same command:

```json
{
  "scripts": {
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "check": "prettier --check src/ && eslint src/"
  }
}
```

Run `npm run check` in CI to fail the build on any formatting or linting violation.

---

## Testing structure

Tests live in one of two locations depending on project style.

*Co-located tests* place a `*.test.js` file next to the file it tests. This makes the
relationship between source and test explicit and keeps related code together. This approach
suits feature-based projects.

*Centralized tests* in a top-level `tests/` or `__tests__/` directory mirror the structure of
`src/`. This approach suits layer-based projects and libraries.

For a feature-based application, co-located tests look like this:

```
src/
├── features/
│   └── tasks/
│       ├── tasks.service.js
│       ├── tasks.service.test.js    # unit test
│       ├── tasks.api.js
│       └── tasks.api.test.js        # integration test
└── shared/
    ├── utils.js
    └── utils.test.js
```

Follow these naming conventions to make test intent clear at a glance:

- `*.test.js`: Unit tests for a single function or module
- `*.spec.js`: Integration or behavior tests for a feature
- `e2e/`: End-to-end (E2E) tests that run against a live environment

---

## Real-world example: task manager application

This section walks through a complete feature-based structure for a browser task manager. The
app allows users to create, complete, and delete tasks, with data persisted to a REST API. The
project runs on Vite with ESLint and Vitest (a Vite-native test runner):

```
task-manager/
├── index.html
├── vite.config.js
├── eslint.config.js
├── .prettierrc
├── .env                         # Local environment variables (not committed)
├── .env.example                 # Placeholder values for other developers
├── package.json
├── public/
│   └── favicon.ico
└── src/
    ├── index.js                 # Entry point: boots the app, registers service worker
    ├── config/
    │   └── index.js             # Reads VITE_API_BASE_URL, validates required values
    ├── features/
    │   ├── auth/
    │   │   ├── auth.service.js
    │   │   ├── auth.service.test.js
    │   │   └── auth.ui.js
    │   └── tasks/
    │       ├── tasks.api.js          # fetch() wrappers for the tasks REST endpoint
    │       ├── tasks.api.test.js
    │       ├── tasks.service.js      # Pure functions: filtering, sorting, validation
    │       ├── tasks.service.test.js
    │       └── tasks.ui.js           # DOM manipulation and event binding
    └── shared/
        ├── error-reporter.js
        └── utils.js
```

The key principle is the three-layer split inside each feature. `tasks.api.js` isolates all
network calls. `tasks.service.js` contains pure functions that transform task data.
`tasks.ui.js` handles all DOM interaction. This separation means you can test the service layer
without a browser or a live network connection:

```js
// src/features/tasks/tasks.service.js

export function filterByStatus(tasks, status) {
    return tasks.filter(task => task.status === status);
}

export function sortByDueDate(tasks) {
    return [...tasks].sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
}

export function isOverdue(task) {
    return task.status !== 'complete' && new Date(task.dueDate) < new Date();
}
```

The corresponding test file confirms each function independently:

```js
// src/features/tasks/tasks.service.test.js

import { describe, it, expect } from 'vitest';
import { filterByStatus, isOverdue } from './tasks.service.js';

describe('filterByStatus', () => {
    it('returns only tasks with the given status', () => {
        const tasks = [
            { id: 1, status: 'open' },
            { id: 2, status: 'complete' },
        ];
        expect(filterByStatus(tasks, 'open')).toEqual([{ id: 1, status: 'open' }]);
    });
});

describe('isOverdue', () => {
    it('returns true when the due date is past and the task is not complete', () => {
        const task = { status: 'open', dueDate: '2000-01-01' };
        expect(isOverdue(task)).toBe(true);
    });

    it('returns false for completed tasks regardless of due date', () => {
        const task = { status: 'complete', dueDate: '2000-01-01' };
        expect(isOverdue(task)).toBe(false);
    });
});
```

---

## Real-world example: sales report script

This example is a Node.js script that fetches sales data from an API, computes weekly totals,
and writes a Markdown report to disk. The script requires no build step and runs directly with
Node.js 18+:

```
sales-report/
├── package.json          # "type": "module" to enable ESM
├── .env                  # API_KEY and REPORT_DIR
├── .gitignore            # .env, node_modules/
└── src/
    ├── index.js          # Orchestrates the full run
    ├── api.js            # fetch() wrappers with error handling
    ├── transform.js      # Pure functions: compute totals, group by week
    └── report.js         # Writes formatted Markdown to disk
```

The entry point calls each module in sequence and exits with a non-zero code on failure:

```js
// src/index.js

import 'dotenv/config';
import { fetchSalesData } from './api.js';
import { groupByWeek, computeTotals } from './transform.js';
import { writeReport } from './report.js';

const { API_KEY, REPORT_DIR } = process.env;

async function main() {
    const raw = await fetchSalesData(API_KEY);
    const weekly = groupByWeek(raw);
    const totals = computeTotals(weekly);
    await writeReport(totals, REPORT_DIR);
    console.log(`Report written to ${REPORT_DIR}/sales-report.md`);
}

main().catch((err) => {
    console.error('Script failed:', err.message);
    process.exit(1);
});
```

The transformation functions in `transform.js` are pure: they accept data and return data with
no network calls or file writes. This makes them straightforward to test with Node's built-in
`node:test` module, with no additional test runner required:

```js
// src/transform.js

export function groupByWeek(records) {
    const weeks = {};
    for (const record of records) {
        const week = getWeekLabel(new Date(record.date));
        weeks[week] ??= [];
        weeks[week].push(record);
    }
    return weeks;
}

export function computeTotals(weeklyGroups) {
    return Object.entries(weeklyGroups).map(([week, records]) => ({
        week,
        total: records.reduce((sum, r) => sum + r.amount, 0),
        count: records.length,
    }));
}

function getWeekLabel(date) {
    const year = date.getFullYear();
    const week = Math.ceil(date.getDate() / 7);
    return `${year}-W${String(week).padStart(2, '0')}`;
}
```

The `??=` operator (logical nullish assignment) initializes `weeks[week]` to an empty array only
if the key does not yet exist. This is a concise alternative to the traditional
`if (!weeks[week]) weeks[week] = []` pattern.

---

## MVC architecture

*Model-View-Controller* (MVC) is a pattern that divides an application into three distinct responsibilities. The *Model* owns the application's data and the rules for changing it. The *View* renders that data to the user and captures input. The *Controller* connects the two: it responds to user actions forwarded by the View, updates the Model, and tells the View to re-render when data changes.

Each layer knows only what it needs to about the others. The View does not know how tasks are stored. The Model does not know what the DOM looks like. Only the Controller holds references to both. This separation means you can test the Model's logic without a browser, change the View's HTML without touching business logic, and swap out the Controller without rewriting either of the other layers.

### Todo app structure

The following example implements a todo app by applying MVC. Each layer lives in its own file:

```
todo-app/
├── index.html
└── src/
    ├── index.js          # Entry point: instantiates and connects all three layers
    ├── model.js          # TaskModel: task data and change notification
    ├── view.js           # TaskView: DOM rendering and event binding
    └── controller.js     # TaskController: responds to user actions, updates Model
```

The HTML provides the structural anchors the View expects. The application mounts to a single root element:

```html
<div id="app">
    <form id="task-form">
        <input id="task-input" type="text" placeholder="New task..." required />
        <button type="submit">Add</button>
    </form>
    <ul id="task-list"></ul>
</div>
<script type="module" src="src/index.js"></script>
```

### The Model

The Model manages all task data. It exposes methods for adding, toggling, and removing tasks. It also implements an *observer pattern*: other layers can register a callback function by calling `subscribe()`. When any method modifies the task list, the Model calls all registered subscribers with the updated data.

The Model never touches the DOM and has no knowledge of the View.

```js
// src/model.js

export class TaskModel {
    #tasks = [];
    #listeners = [];

    add(text) {
        const task = { id: Date.now(), text, complete: false };
        this.#tasks.push(task);
        this.#notify();
    }

    toggle(id) {
        const task = this.#tasks.find(t => t.id === id);
        if (task) {
            task.complete = !task.complete;
            this.#notify();
        }
    }

    remove(id) {
        this.#tasks = this.#tasks.filter(t => t.id !== id);
        this.#notify();
    }

    getAll() {
        return [...this.#tasks];    // return a copy to prevent external mutation
    }

    subscribe(listener) {
        this.#listeners.push(listener);
    }

    #notify() {
        this.#listeners.forEach(listener => listener(this.getAll()));
    }
}
```

`getAll()` returns a shallow copy of the internal array. This prevents outside code from accidentally mutating the Model's private state by modifying the returned array directly.

`#notify()` calls every registered listener with a fresh copy of the tasks. At the time the Model is created, no subscribers exist. The Controller registers one in a later step.

### The View

The View handles two responsibilities: rendering task data to the DOM, and exposing *binding methods* that the Controller calls to attach event handlers. The View never calls Model methods directly. Instead, it calls whatever function was passed to its `bind*` methods.

This design applies *dependency inversion*. The View does not depend on a specific Controller. It depends only on the contract: "when the user adds a task, call the function you were given."

```js
// src/view.js

export class TaskView {
    #form;
    #input;
    #list;

    constructor(root) {
        this.#form  = root.querySelector('#task-form');
        this.#input = root.querySelector('#task-input');
        this.#list  = root.querySelector('#task-list');
    }

    // The Controller calls this to attach its add-task handler
    bindAddTask(handler) {
        this.#form.addEventListener('submit', (e) => {
            e.preventDefault();
            const text = this.#input.value.trim();
            if (text) {
                handler(text);
                this.#input.value = '';
            }
        });
    }

    // The Controller calls this to attach its toggle handler.
    // Event delegation: one listener on the list handles all toggle clicks.
    bindToggleTask(handler) {
        this.#list.addEventListener('click', (e) => {
            if (e.target.matches('.toggle')) {
                handler(Number(e.target.dataset.id));
            }
        });
    }

    // The Controller calls this to attach its remove handler
    bindRemoveTask(handler) {
        this.#list.addEventListener('click', (e) => {
            if (e.target.matches('.remove')) {
                handler(Number(e.target.dataset.id));
            }
        });
    }

    // The Controller calls this when the Model data changes
    render(tasks) {
        this.#list.innerHTML = tasks.map(task => `
            <li class="${task.complete ? 'complete' : ''}">
                <span>${task.text}</span>
                <button class="toggle" data-id="${task.id}">
                    ${task.complete ? 'Undo' : 'Complete'}
                </button>
                <button class="remove" data-id="${task.id}">Delete</button>
            </li>
        `).join('');
    }
}
```

The toggle and remove listeners attach to the `<ul>` element rather than to individual buttons. This is *event delegation*. Because `render()` replaces the entire list's `innerHTML` on every update, buttons created during a previous render are destroyed and recreated with each change. Attaching listeners to the stable parent element avoids losing them on every re-render.

### The Controller

The Controller receives the Model and View as constructor arguments. It creates nothing itself. Its constructor does two things:

1. Calls each `bind*` method on the View, passing its own handler methods as callbacks.
2. Subscribes the View's `render` method to Model change notifications.

After the constructor runs, the three layers are fully connected. The Controller does not need to call `render` explicitly at any later point.

```js
// src/controller.js

export class TaskController {
    #model;
    #view;

    constructor(model, view) {
        this.#model = model;
        this.#view  = view;

        // Step 1: register Controller handlers with the View
        this.#view.bindAddTask(this.#handleAddTask.bind(this));
        this.#view.bindToggleTask(this.#handleToggleTask.bind(this));
        this.#view.bindRemoveTask(this.#handleRemoveTask.bind(this));

        // Step 2: subscribe the View's render method to Model changes
        this.#model.subscribe(tasks => this.#view.render(tasks));
    }

    #handleAddTask(text) {
        this.#model.add(text);
    }

    #handleToggleTask(id) {
        this.#model.toggle(id);
    }

    #handleRemoveTask(id) {
        this.#model.remove(id);
    }
}
```

The `.bind(this)` call on each handler is necessary because the View stores and calls those functions later, from a different execution context. Without `.bind(this)`, `this` inside the handler would be `undefined` in strict mode, and the private fields `#model` and `#view` would be inaccessible. An arrow function wrapper like `(text) => this.#handleAddTask(text)` is an equivalent alternative.

### Wiring the layers together

The entry point instantiates the Model, instantiates the View, and passes both into the Controller. The Controller's constructor handles all binding:

```js
// src/index.js

import { TaskModel }      from './model.js';
import { TaskView }       from './view.js';
import { TaskController } from './controller.js';

const model = new TaskModel();
const view  = new TaskView(document.querySelector('#app'));

new TaskController(model, view);
```

The entry point never stores a reference to the Controller. Once the constructor runs, the event listeners and subscriptions hold the connections in place for the lifetime of the page.

### Data flow

Every user interaction follows the same path through the layers:

1. The user takes an action: submits the form, clicks **Complete**, or clicks **Delete**.
2. The View captures the DOM event and calls the bound handler.
3. The Controller handler receives the input and calls the appropriate Model method.
4. The Model updates its internal data and calls `#notify()`.
5. `#notify()` calls every registered subscriber — in this case, `view.render(tasks)`.
6. The View re-renders the list with the updated data.

No step skips a layer. The View never calls Model methods directly. The Model never calls View methods directly. All coordination passes through the Controller.
