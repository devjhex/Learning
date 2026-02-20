1️⃣ What is a task?
- A task is a discrete unit of work that has to be executed on the main thred. 

2️⃣ Where are tasks stored?
- Taks are stored in the task queue and micro task queue.

3️⃣ When does a task run?
- A task runs when the callstack is empty, the micro tasks have been executed and posssibly a render if needed, if all of that is done then a macro task is run.

4️⃣ What creates tasks? (e.g., events, timers, parsing)
- **Event handlers:** `click`, `scroll`, `resize`, `load`, etc.
- **Timers:** `setTimeout()`, `setInterval()`, `setImmediate()`
- **Parsing:** HTML/CSS parsing, script execution
- **I/O operations:** Network requests, file reads
- **`requestAnimationFrame()`** callbacks
- **`scheduler.postTask()`** - explicit task scheduling

For **Microtasks** (since they’re tied to scheduling):

1️⃣ What is a microtask?
- A microtask is a high priority callback.

2️⃣ When does it run relative to a task?
- All micro tasks in the micro task queue are run after each task from the macro task queue.

3️⃣ What creates microtasks? (e.g., promises)
- lines of code after the await keyword in an async function
- code inside a then of a promise
- queueMicrotask()
- Mutation Observer

For **Scheduling**:

1️⃣ What does scheduling mean in the browser context?
- Scheduling means controlling in when and in what order some tasks are executed on the main thread.

2️⃣ Who decides what runs next?

- **The Event Loop** decides what runs next based on a strict priority order:
  1. **Callstack** - Any synchronous code currently executing
  2. **Microtask Queue** - All microtasks (promises, queueMicrotask, mutation observers)
  3. **Rendering** - Layout, paint, composite (if needed)
  4. **Macrotask Queue** - One macrotask (setTimeout, events, I/O)
  5. Loop back to step 1
NOTE:

- The browser follows this algorithm automatically - you don't control it directly, but you influence it by scheduling tasks with `setTimeout()`, `scheduler.postTask()`, or letting promises queue microtasks.

3️⃣ What is the order between task → microtask → render?
- **Macrotask (Task) → Microtasks → Render → Macrotask → Microtasks → Render** (repeat)

**The Cycle:**
1. Execute ONE macrotask
2. Execute ALL microtasks in the queue
3. Render (if needed) - layout, paint, composite
4. Go back to step 1 (pick next macrotask)

**Key points:**
- Only ONE macrotask runs per cycle
- ALL microtasks run before rendering
- Render only happens if there were style/layout changes and enough time has passed
- If microtasks queue new microtasks, they run before rendering
- `scheduler.yield()` prioritizes your continuation (runs sooner in next cycle)