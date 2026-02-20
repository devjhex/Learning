# Tasks & Scheduling in JavaScript

## Task

**Definition:** A discrete unit of work the browser executes on the main thread. Any task longer than 50ms is considered a "long task."

**Examples:**
- Parsing HTML/CSS
- Executing JavaScript
- Event handlers
- Rendering/painting

**Run-to-Completion Model:** Each task runs until it finishes before the next task begins, blocking interactions during execution.

---

## Scheduling

**Definition:** The process of controlling when and in what order tasks execute on the main thread, and yielding execution to allow the browser to process higher-priority work (user interactions, rendering).

**Key Concept:** Break long tasks into smaller chunks and yield to let the browser handle critical work (layout, paint, user input) between chunks.

---

## Task Queue

**Definition:** The queue where tasks wait to be executed on the main thread. When you yield, your task gets added to the queue. With `scheduler.yield()`, your continuation is prioritized; with `setTimeout()`, it goes to the end.

---

## Yielding

**Definition:** Intentionally pausing execution to give the main thread an opportunity to process higher-priority work before resuming.

**Methods:**
- `setTimeout(fn, 0)` - Basic yielding (continuation goes to end of queue)
- `scheduler.yield()` - Prioritized yielding (continuation runs before other tasks)

---

## Key Principle

**Don't block the main thread.** Tasks must not exceed ~50ms. Yield strategically to prioritize user-facing work (interactions, rendering) over background work (analytics, data processing).

**Where tasks are stored:** Tasks are stored in the **Task Queue** (also called the **Macrotask Queue** or **Event Queue**) managed by the browser's event loop. This is separate from the **Microtask Queue** (where promises, `queueMicrotask()`, and mutation observers go). The event loop processes all microtasks before moving to the next macrotask.
