# 🐱 Magic Cat Engine

> **The builder that builds builders.** A zero-dependency, single-file visual application builder — open `magiccatengine.html` in any browser and start building.

Magic Cat Engine is a machine-based, no-code / low-code interface for constructing HTML/JavaScript applications visually. You build with **Machines** (components), wire them to **Pipes** (data), **Events** (messaging), and **Views** (pages), then export a fully working standalone HTML app — or export the engine's own state as JSON so another instance can boot from it exactly.

---

## Table of Contents

- [The Core Idea](#the-core-idea)
- [Getting Started](#getting-started)
- [Interface Overview](#interface-overview)
  - [Left Panel](#left-panel)
  - [Canvas](#canvas)
  - [Right Inspector](#right-inspector)
  - [Center Modes](#center-modes)
  - [Cat Log](#cat-log)
- [The Four Primitives](#the-four-primitives)
  - [Machines](#machines)
  - [Pipes](#pipes)
  - [Events](#events)
  - [Views](#views)
- [The Global Namespace Rule](#the-global-namespace-rule)
- [Wiring Machines](#wiring-machines)
- [The Database](#the-database)
- [View HTML & Data Binding Attributes](#view-html--data-binding-attributes)
- [CSS Inspector](#css-inspector)
- [Export HTML — Building a Real App](#export-html--building-a-real-app)
- [Export JSON & Import](#export-json--import)
- [Self-Replication](#self-replication)
- [The .mce.json Format](#the-mcejson-format)
- [Example Files](#example-files)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [The Machine Constant Concept](#the-machine-constant-concept)
- [License](#license)

---

## The Core Idea

Most builders treat a UI as a tree of visual elements. Magic Cat Engine treats it as a network of **machines** connected by **named channels**. Every machine is a node. Every pipe, event, and view is a public, globally named channel any machine can connect to. You wire things together through a visual interface — no code required for the wiring itself.

The key insight is the **machine constant**: every object in the system can describe itself completely as JSON. Because of this, the engine can export your whole application — including its own builder — as a self-describing document that any MCE instance can load and run.

---

## Getting Started

1. Download `magiccatengine.html` from this repository.
2. Open it in any modern browser (Chrome, Firefox, Edge, Safari). No server needed.
3. The engine boots with a demo setup — one pipe, one event, one view, and two machines already wired together.
4. Click **Preview App** in the header to see the live rendered output immediately.
5. Click **Export HTML ↗** to download a fully working standalone app.

That's it. No npm, no build step, no dependencies.

---

## Interface Overview

The interface is divided into five zones:

```
┌─────────────────────────────────────────────────────────────────┐
│  Header — mode buttons, export, import, replicate               │
├──────────┬──────────────────────────────────────┬───────────────┤
│          │                                      │               │
│  Left    │           Center                     │   Right       │
│  Panel   │   Canvas / Preview / HTML Source     │   Inspector   │
│          │                                      │               │
│  DB      │                                      │   Wire        │
│  Pipes   │                                      │   onEvent     │
│  Events  │                                      │   CSS         │
│  Views   │                                      │   Data        │
│          │                                      │   Code        │
├──────────┴──────────────────────────────────────┴───────────────┤
│  DOM Tree bar                                                    │
├─────────────────────────────────────────────────────────────────┤
│  Cat Log                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Left Panel

Four tabs manage all global resources:

**DB** — Write and read JSON documents to a session-scoped in-memory database. Choose a collection name, paste a JSON document, and hit Write. The Global Registry at the bottom shows every currently reserved name across Pipes, Events, and Views.

**Pipes** — Create named data channels. Each pipe has a direction (Input = DB→Screen, Output = Screen→DB) and a collection name. Click a pipe while a machine is selected to wire it instantly.

**Events** — Create named pub/sub events. Each event has a name and a payload key schema (e.g. `id, label, value`). Events are fired by machines and received by any machine that listens for them.

**Views** — Create named HTML pages. Each view is bound to a URL variable (`?view=name`) and has a template you write in the HTML Source editor. Double-click a view in the list to jump straight to the preview.

### Canvas

The main build surface. Machines appear as draggable cards.

- **Drag** a machine by its header bar to reposition it.
- **Click** a machine to select it and focus the Inspector.
- **SVG wires** draw bezier curves between machines that share a wired pipe or view — showing the data topology at a glance.
- The **DOM Tree bar** below the canvas shows the internal structure of the selected machine as breadcrumb nodes. Click any node to target it in the CSS inspector.

### Right Inspector

Five tabs, focused on the selected machine:

| Tab | Purpose |
|-----|---------|
| **Wire** | Connect pipes and views to this machine. Click an available pipe or view to wire it; remove wires with ✕ |
| **onEvent** | Toggle which global events this machine listens for. Wire a `do:` instruction (update display, route to view, etc.) with a payload key path |
| **CSS** | Apply CSS to any DOM node inside the machine. Target a specific node by clicking a crumb in the DOM Tree bar |
| **Data** | Bind an input pipe's collection and field to a render target selector |
| **Code** | View the machine's full serialized JSON constant. Copy to clipboard or duplicate the machine |

### Center Modes

The center panel has three modes, switched from the mode bar:

**🗂 Canvas** — The visual build surface described above.

**👁 View Preview** — A live sandboxed iframe rendering of any view. Select a view from the dropdown, or type `?view=name` in the address bar. The MCE runtime is injected into the preview, so event binding, pipe data, and the router all work. This is what your users will actually see.

**📝 HTML Source** — A code editor for the selected view's HTML template. Write any HTML. Use [data binding attributes](#view-html--data-binding-attributes) to wire data and events declaratively. Hit **Save** then **Preview ↗** to see it instantly.

### Cat Log

A filterable log panel at the bottom of the screen. Every engine action — writes, reads, event fires, wire connections, exports — streams through here in real time. Filter by INFO / OK / WARN / ERR / EVT.

---

## The Four Primitives

### Machines

A machine is the fundamental unit of Magic Cat Engine. It maps to a DOM element or component in your app. Each machine has:

- A **name** (display label)
- A **type** (Display, Input, Button, List, Transform, Router, Emitter, Collector)
- **Input and output ports** (visual connection points)
- A list of **wires** connecting it to pipes, events, and views
- **CSS state** per DOM node
- A **data binding** (which pipe field to display, and where)
- **Event listeners** and **instructions**

In the `.mce.json` format, machines are represented as JSON objects with a `tag` (the HTML element type), `attrs`, `text`, `children`, `wires`, `pipeBindings`, `css`, and a `logic` block.

### Pipes

Pipes are named data channels between the database and machines.

- **Input pipe** — reads documents from a DB collection and pushes data into wired machines
- **Output pipe** — receives data from a machine and writes it to a DB collection

Pipes are declared once and can be wired to any number of machines. A machine with an input pipe wired to it can pull its bound field from the latest document in that collection.

**Important:** Pipe names are globally unique and shared with Events and Views in a single namespace.

### Events

Events are named pub/sub signals with a payload schema.

- Any machine can **emit** an event by clicking the `emit` button on its card, or via an `onclick` attribute in view HTML (`EventBus.emit('eventName', payload)`)
- Any machine can **listen** for any event via the onEvent tab in the Inspector
- When an event fires, all listening machines execute their wired instruction

The event system is the backbone of inter-machine communication. Because all events are named and global, any machine anywhere in the app can react to anything that happens anywhere else.

**Important:** Event names are globally unique and shared with Pipes and Views.

### Views

Views are named HTML pages rendered by the MCE router.

- Each view has a URL variable: `?view=viewName`
- The HTML template is written in the HTML Source editor
- Views are rendered into a target element (e.g. `#main`, `body`)
- In the exported app, views are navigated via a nav bar and the URL hash (`#view=viewName`)

Views are the screens your users interact with. Machines can be wired to a view so that emitting the machine routes the app to that view.

**Important:** View names are globally unique and shared with Pipes and Events.

---

## The Global Namespace Rule

**Pipes, Events, and Views all share one global name registry. No two of them can have the same name.**

This is intentional and fundamental to how the engine works. Because all three are public channels that any machine can connect to, they must be unambiguously addressable by name alone. When you call `EventBus.emit('myName', ...)` or `?view=myName`, there is exactly one thing in the system with that name.

The engine enforces this at creation time — if you try to create a pipe named `submit` and an event named `submit` already exists, you will get an error.

The **Global Registry** in the DB tab shows every reserved name at a glance, color-coded by type.

---

## Wiring Machines

Wiring is the act of connecting a machine to a global channel (pipe, event, or view).

**To wire a pipe to a machine:**
1. Select a machine on the canvas (click it)
2. In the left panel, switch to the Pipes tab
3. Click any pipe in the list — it wires instantly
4. Or use the Wire tab in the Inspector and click from the Available Pipes section

**To wire an event listener:**
1. Select a machine
2. Go to the onEvent tab in the Inspector
3. Click any event row to toggle it ON — the machine now listens for that event
4. Choose a `do:` instruction and optional payload key path, then click Wire Instruction

**To wire a view:**
1. Select a machine
2. In the Inspector Wire tab, click any view in the Available Views section
3. Now clicking `view` on the machine card (or emitting from it with a route instruction) will navigate to that view

SVG bezier curves on the canvas connect machines that share the same wired channel, so you can see the data flow topology at a glance.

---

## The Database

The database is **session-scoped and in-memory**. It resets when the page refreshes. This is by design for the builder — it keeps the environment clean.

In the DB tab:

- Set a **Collection** name (like `users`, `items`, `products`)
- Write a **JSON document** to that collection
- Read the collection to see all documents
- Preview shows the current collection contents

In the exported app, the same `MCE_DB` runtime is available. Data written during a session persists for that session. For persistent storage, connect a backend API to your view's HTML template.

---

## View HTML & Data Binding Attributes

When writing a view template in the HTML Source editor, you can wire data and events declaratively with `data-mce-*` attributes. The MCE runtime processes these automatically when a view is rendered.

### `data-mce-pipe`
Reads the latest document from a named pipe's collection and sets the element's `textContent` to the value of the bound field.

```html
<span data-mce-pipe="userFeed" data-mce-field="name">Loading...</span>
```

### `data-mce-field`
Specifies which field of the document to display. Works alongside `data-mce-pipe` and `data-mce-list`.

```html
<h2 data-mce-pipe="productFeed" data-mce-field="title"></h2>
```

### `data-mce-event`
Wires a click on this element to emit the named event, passing `{ el, ts }` as the payload.

```html
<button data-mce-event="onSubmitClick">Submit</button>
```

### `data-mce-list`
Renders all documents from a named pipe's collection as `<li>` elements inside this element.

```html
<ul data-mce-list="itemsFeed" data-mce-field="label"></ul>
```

### `data-mce-machine`
Marks an element as the display target for a named machine. When a machine receives an `update display` instruction, it sets the `textContent` of all elements with its name here.

```html
<div data-mce-machine="StatusDisplay"></div>
```

### Manual event wiring in templates

You can also use the `MCE_BUS` object directly in inline scripts or `onclick` attributes:

```html
<button onclick="MCE_BUS.emit('onCartUpdate', { item: 'apple', qty: 1 })">
  Add to Cart
</button>

<script>
  MCE_BUS.on('onCartUpdate', function(payload) {
    document.getElementById('cart-count').textContent = payload.qty;
  });
</script>
```

---

## CSS Inspector

The CSS tab in the Inspector lets you style any DOM node inside the selected machine.

1. Select a machine on the canvas
2. In the DOM Tree bar (below the canvas), click a crumb to target a specific node — `.machine`, `.mhead`, `.mname`, `.mbody`, `.mfoot`, etc.
3. Switch to the CSS tab in the Inspector
4. Fill in any CSS property values
5. Click **Apply CSS** — the styles apply live to the machine card

CSS state is saved per machine per DOM node and is included in the machine's JSON constant, so it persists across Export JSON / Import JSON cycles.

---

## Export HTML — Building a Real App

**Export HTML ↗** (green button, or `Ctrl+E`) generates a fully self-contained `mce-app.html` file.

The exported file contains:

- A navigation bar linking to all your views
- URL hash routing (`#view=viewName`) with `popstate` support
- All view HTML templates rendered on demand
- The full **MCE Runtime** baked in:
  - `MCE_DB` — in-memory document store
  - `MCE_BUS` — pub/sub event bus with `emit` and `on`
  - `MCE_PIPES` — pipe read/write wired to collections
  - `MCE_router` — view router with `go(viewName)` and `boot()`
  - All machine event listener wiring from your build
  - All `data-mce-*` attribute bindings auto-applied on view render
- A `replicateSelf()` function that exports the app's own machine constant JSON

No external libraries. No CDN calls. One file. Works offline.

**Example — calling the runtime from a view template:**

```html
<!-- In your view HTML template -->
<button onclick="MCE_BUS.emit('onLogin', { user: 'alice' })">Log In</button>
<div id="status"></div>

<script>
  MCE_BUS.on('onLogin', function(p) {
    document.getElementById('status').textContent = 'Welcome, ' + p.user;
    MCE_router.go('dashboardView');
  });
</script>
```

---

## Export JSON & Import

**Export JSON** saves the entire engine state — all pipes, events, views, machines, and their wiring — as a `.json` file. This is the machine constant of your current session.

**Import JSON** loads a previously exported `.json` file back into the engine, merging its pipes, events, views, and machines onto the canvas without overwriting anything already there (deduplication by `id`).

The JSON schema:

```json
{
  "_engine": "MagicCatEngine",
  "version": "1.1.0",
  "exported": "2026-05-28T00:00:00.000Z",
  "pipes": [ { "id": "mc_1", "name": "dataFeed", "dir": "input", "collection": "items" } ],
  "events": [ { "id": "mc_2", "name": "onItemClick", "payload": "id, label" } ],
  "views": [ { "id": "mc_3", "name": "mainView", "url": "?view=mainView", "target": "#main", "template": "<div>...</div>" } ],
  "machines": [
    {
      "id": "mc_4", "name": "DataDisplay", "type": "display",
      "icon": "📺", "color": "#e8a020",
      "x": 60, "y": 50,
      "wires": [ { "kind": "pipe", "rid": "mc_1", "name": "dataFeed" } ],
      "onEvents": [],
      "binding": { "pipe": "dataFeed", "collection": "items", "field": "label", "target": ".mname" },
      "css": {},
      "instructions": []
    }
  ],
  "db_collections": ["items"]
}
```

---

## Self-Replication

The **Replicate Self** button runs a deliberate self-description sequence and then exports the engine's own state as JSON. The key concept:

> The engine is not copying its own source code. It is **transmitting its individuality** — the current configuration of all machines, pipes, events, views, and their wiring — as a portable machine constant document.

A fresh MCE instance that imports this JSON will reconstruct the same application state exactly. This is the same principle that allows the exported app to call `replicateSelf()` — the app knows how to describe itself back.

This concept extends beyond HTML/JavaScript. The machine constant is a language-agnostic description. The same JSON schema could be used to generate equivalent applications in Python, Swift, C++, or any environment that implements an MCE runtime.

---

## The .mce.json Format

The example files in this repository use an extended `.mce.json` schema (version 1.0.0) that represents machines as actual DOM element nodes with parent/child relationships, forming a real HTML tree. This is the canonical format for sharing MCE projects.

Key differences from the engine's internal state JSON:

| Field | Description |
|-------|-------------|
| `rootOrder` | Array of top-level machine IDs in render order |
| `machines` | Object map of `id → machine`, not an array |
| `machine.tag` | The HTML element type (`div`, `button`, `h1`, etc.) |
| `machine.attrs` | HTML attributes, including `onclick` for event wiring |
| `machine.text` | Inner text content |
| `machine.children` | Array of child machine IDs |
| `machine.parentId` | Parent machine ID or `null` for root machines |
| `machine.wires` | Array of `{ eventName, action, actionArgs }` — event→action wiring |
| `machine.pipeBindings` | Pipe data bindings for this element |
| `machine.viewBinding` | Which view this machine routes to |
| `machine.css` | CSS properties object |
| `logic` | Per-machine logic blocks (future extension) |
| `events` | Object map of event definitions |
| `pipes` | Object map of pipe definitions |
| `views` | Object map of view definitions |
| `dbCollections` | Array of collection names used |
| `_nextId` | ID counter for the next machine |

**Wire action types:**

| Action | Effect |
|--------|--------|
| `toggle` | Toggle element visibility |
| `show` | Show element |
| `hide` | Hide element |
| `setText` | Set element text content from payload |
| `addClass` | Add a CSS class |
| `removeClass` | Remove a CSS class |
| `navigate` | Route to a view |
| `pipeRead` | Pull data from a pipe |
| `pipeWrite` | Push data to a pipe |

---

## Example Files

The repository includes 29 `.mce.json` examples covering the full range of engine capabilities. Import any of them into the engine via **Import JSON**.

| File | Demonstrates |
|------|-------------|
| `01-toggle-card.mce.json` | Event wiring, toggle action |
| `02-tabs-ui.mce.json` | Tab switching with show/hide wires |
| `03-accordion.mce.json` | Accordion panels, event-driven visibility |
| `04-notification-toast.mce.json` | Temporary toast messages via events |
| `05-wizard-stepper.mce.json` | Multi-step wizard with state |
| `06-guest-book.mce.json` | Form submission writing to a DB collection |
| `07-trivia-quiz.mce.json` | Interactive quiz with scoring |
| `08-staff-directory.mce.json` | List rendering from a pipe |
| `09-feature-requests.mce.json` | User submission + list display |
| `10-url-views.mce.json` | Multi-view routing with `?view=` |
| `11-team-dashboard.mce.json` | Dashboard with multiple data feeds |
| `12-design-studio.mce.json` | Live CSS editing via events |
| `13-startup-hub.mce.json` | Multi-section marketing page |
| `14-self-replicating-machine.mce.json` | Machine that exports its own JSON |
| `15-machine-builder.mce.json` | MCE building MCE |
| `16-no-code-data-flow.mce.json` | Pipe flow without writing code |
| `17-transfer-value.mce.json` | Value passing between machines |
| `18-visibility-actions.mce.json` | Show/hide/toggle patterns |
| `19-live-text-filters.mce.json` | Real-time text filtering |
| `20-multi-view-navigation.mce.json` | Full multi-page app navigation |
| `21-event-chain-relay.mce.json` | Chained events across machines |
| `22-live-html-preview.mce.json` | Live rendering of HTML input |
| `23-mock-website.mce.json` | Complete mock multi-page website |
| `24-full-feature-demo.mce.json` | Kitchen-sink demonstration |
| `25-no-code-mini-site.mce.json` | Complete site with zero code |
| `26-self-hosting-mini-ide.mce.json` | Mini IDE built inside MCE |
| `27-codeless-html-css-builder.mce.json` | Drag-and-drop style builder |
| `28-codeless-profile-state.mce.json` | User profile with state management |
| `29-codeless-db-loop-catalog.mce.json` | Product catalog from DB loop |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl / Cmd + M` | Open Add Machine dialog |
| `Ctrl / Cmd + E` | Export HTML |
| `Delete` | Delete selected machine (when not in a text field) |
| `Escape` | Close any open modal |
| `Enter` | Confirm modal (when not in a textarea) |

---

## The Machine Constant Concept

The machine constant is the philosophical core of Magic Cat Engine.

Every object in a program — a button, a data feed, a page, an event — can be described completely as a named, structured document. That document is the machine's **constant**: an invariant description of what it is, what it connects to, and how it behaves.

Because these constants are data, not code, they can be:

- **Transmitted** — sent between systems, stored in a database, emailed
- **Versioned** — diffed, merged, rolled back like any document
- **Translated** — the same JSON constant could generate an iOS view, a React component, or a C++ class
- **Self-describing** — an instance of the engine that reads its own constant can reconstruct itself

This is why MCE is described as "the builder that builds builders." The engine itself is a set of machine constants. Export the engine's state and you have a document that is both *the application* and *the description of how to rebuild the application*.

The concept is language-agnostic. A Python MCE runtime, a Swift MCE runtime, or a C++ MCE runtime would all consume the same `.mce.json` files and produce equivalent behavior in their respective environments.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

*Magic Cat Engine is built and maintained by [MagicCatSoftware](https://github.com/MagicCatSoftware).*
