# The Main thread
## Introduction

Common advice for keeping JavaScript apps fast boils down to:
- "Don't block the main thread"
- "Break up your long tasks"

This article explains what this means, how the browser handles tasks, and practical strategies to implement task optimization.

---

## What is a Task?

A **task** is any discrete piece of work that the browser does, including:
- Rendering
- Parsing HTML and CSS
- Running JavaScript
- Other types of work

JavaScript you write is perhaps the largest source of tasks. Tasks associated with JavaScript impact performance in two main ways:

1. **During startup:** When a browser downloads a JavaScript file, it queues tasks to parse and compile that JavaScript for later execution
2. **During page lifecycle:** Tasks are queued when JavaScript responds to interactions (event handlers), JavaScript-driven animations, and background activity (analytics collection)

All of this happens on the **main thread** (with the exception of web workers and similar APIs).

---

## What is the Main Thread?

The **main thread** is where most tasks run in the browser and where almost all JavaScript code is executed.

### Key Characteristics:

- **Single-threaded processing:** The main thread can only process one task at a time
- **Long task definition:** Any task that takes longer than **50 milliseconds** is considered a long task
- **Blocking period:** For tasks exceeding 50ms, the "blocking period" is the task's total time minus 50 milliseconds

### User Experience Impact:

The browser blocks interactions while a task is running, but this isn't noticeable to users as long as tasks don't run too long. However, when there are many long tasks:
- The user interface feels unresponsive
- The page may feel broken if the main thread is blocked for very long periods

### Breaking Up Tasks:

Breaking a long task into several smaller ones is crucial because:
- **The browser can respond to higher-priority work sooner**, including user interactions
- After a task yields, remaining tasks then run to completion, ensuring the initial work gets done
- User interactions can begin earlier rather than waiting for an entire long task to complete

**Visual Effect:** When a user interacts with a page and an event handler is queued during a long task, that handler must wait. If the task is broken into smaller chunks, the event handler can begin much sooner, making the interaction feel instant rather than laggy.

---

## Task Management Strategies

### 1. Manually Defer Code Execution with `setTimeout()`

**Purpose:** Yield to the main thread by briefly interrupting work to give the browser opportunities to run more important tasks.

**How it works:** Pass a function to `setTimeout()` with a timeout of `0`. This postpones execution into a separate task.

**Example:**
```javascript
function saveSettings () {
  // Do critical work that is user-visible:
  validateForm();
  showSpinner();
  updateUI();

  // Defer work that isn't user-visible to a separate task:
  setTimeout(() => {
    saveToDatabase();
    sendAnalytics();
  }, 0);
}
```

**Key Point:** Yield to the main thread when you have user-facing work (like UI updates) that should execute sooner than if you didn't yield.

**Drawbacks of `setTimeout()`:**

1. **Developer ergonomics:** Not ideal for large amounts of data that need processing in a loop
2. **Nested timeout delays:** After five rounds of nested `setTimeout()` calls, the browser imposes a minimum 5-millisecond delay for each additional call
3. **Task queue position:** When you defer code using `setTimeout()`, the task gets added to the end of the queue. If other tasks are waiting, they will run before your deferred code

### 2. Breaking Up Functions (Not Effective Alone)

While breaking work into smaller functions is good software architecture, **JavaScript runs all called functions as a single task** because they execute within the parent function's context.

**Example Problem:**
```javascript
function saveSettings () {
  validateForm();
  showSpinner();
  saveToDatabase();
  updateUI();
  sendAnalytics();
}
```

All five functions run as **one long task**, so the UI won't show any response until all functions complete. This is because JavaScript uses the **run-to-completion model** of task execution—each task runs until it finishes, regardless of how long it blocks the main thread.

---

## A Dedicated Yielding API: `scheduler.yield()`

### What is `scheduler.yield()`?

`scheduler.yield()` is an API **specifically designed for yielding to the main thread** in the browser.

- It's a function that returns a `Promise` that resolves in a future task
- Any code chained after the promise (via `.then()` or `await`) runs in that future task
- Simply insert `await scheduler.yield()` to pause execution and yield to the main thread

**Example:**
```javascript
async function saveSettings () {
  // Do critical work that is user-visible:
  validateForm();
  showSpinner();
  updateUI();

  // Yield to the main thread:
  await scheduler.yield()

  // Work that isn't user-visible, continued in a separate task:
  saveToDatabase();
  sendAnalytics();
}
```

**Result:** The function executes in two separate tasks. Between them, the browser can run layout and paint work, giving users a quicker visual response.

### Key Advantages Over `setTimeout()`:

1. **Prioritized continuation:** The continuation of a yielded function runs before other similar tasks start, avoiding interruption by third-party task sources
2. **Ergonomic:** Works seamlessly with `async`/`await` syntax
3. **Better control:** Don't have to yield after every function call—only between critical and background work

### Cross-Browser Support

`scheduler.yield()` is not yet supported in all browsers, so a fallback is needed.

**Option 1: Use a polyfill**
```javascript
// Install scheduler-polyfill and use directly
import 'scheduler-polyfill';
```

**Option 2: Simple fallback with Promise and setTimeout**
```javascript
function yieldToMain () {
  if (globalThis.scheduler?.yield) {
    return scheduler.yield();
  }

  // Fall back to yielding with setTimeout.
  return new Promise(resolve => {
    setTimeout(resolve, 0);
  });
}
```

**Important:** Check for both the global `scheduler` object and that it has a `yield` method.

**Option 3: Progressive enhancement (one-liner)**
```javascript
// Yield to the main thread if scheduler.yield() is available.
await globalThis.scheduler?.yield?.();
```

### Browser Support for `scheduler.yield()`:
- **Chrome:** 129+
- **Edge:** 129+
- **Firefox:** 142+
- **Safari:** Not supported (requires fallback)

### Breaking Up Long-Running Work with `scheduler.yield()`

**Basic approach:**
```javascript
async function runJobs(jobQueue) {
  for (const job of jobQueue) {
    // Run the job:
    job();

    // Yield to the main thread:
    await yieldToMain();
  }
}
```

**Problem:** If jobs are very short, yielding overhead can accumulate and take more time than the actual work.

**Optimized approach: Batch jobs with deadline-based yielding**
```javascript
async function runJobs(jobQueue, deadline=50) {
  let lastYield = performance.now();

  for (const job of jobQueue) {
    // Run the job:
    job();

    // If it's been longer than the deadline, yield to the main thread:
    if (performance.now() - lastYield > deadline) {
      await yieldToMain();
      lastYield = performance.now();
    }
  }
}
```

**Benefits:** 
- Jobs are broken up to never take too long to run
- The runner only yields about every 50 milliseconds
- Balances responsiveness with completion time

**Important Notes:**
- `scheduler.yield()` only schedules a task; the `await` actually pauses execution
- Don't forget the `await` keyword
- Avoid array methods like `Array.prototype.forEach()` which don't wait for each callback to resolve before moving to the next iteration

### Yielding Without Continuation vs. With Continuation

This is a critical difference that affects how your code executes:

#### WITHOUT Continuation (using `setTimeout()`)

When you yield with `setTimeout()`, your continuation (remaining code) goes to the **end of the task queue**:

```
Task Queue:
┌─────────────────────┐
│ Your Task (Part 1)  │  ← Runs first, then yields
├─────────────────────┤
│ Other Task #1       │  ← These run while your code waits
├─────────────────────┤
│ Other Task #2       │
├─────────────────────┤
│ Your Task (Part 2)  │  ← Your continuation runs LAST
└─────────────────────┘
```

**Problem:** Third-party tasks can interrupt your code's execution order.

#### WITH Continuation (using `scheduler.yield()`)

When you yield with `scheduler.yield()`, your continuation is **prioritized** and runs before other similar tasks:

```
Task Queue:
┌─────────────────────┐
│ Your Task (Part 1)  │  ← Runs first, then yields
├─────────────────────┤
│ Your Task (Part 2)  │  ← Your continuation is prioritized (runs next)
├─────────────────────┤
│ Other Task #1       │  ← Third-party tasks run after
├─────────────────────┤
│ Other Task #2       │
└─────────────────────┘
```

**Benefit:** Your code's execution order is preserved; third-party tasks won't interrupt your work.

#### Practical Difference in Code

**Without continuation (setTimeout):**
```javascript
async function processWithoutContinuation() {
  console.log("Step 1: Start");
  
  // Yield with setTimeout
  await new Promise(resolve => setTimeout(resolve, 0));
  
  console.log("Step 3: Continuation (runs AFTER other tasks)");
}

// Other code from third-party library
thirdPartyCode();  // Might run BEFORE your continuation
```

**With continuation (scheduler.yield):**
```javascript
async function processWithContinuation() {
  console.log("Step 1: Start");
  
  // Yield with scheduler.yield()
  await scheduler.yield();
  
  console.log("Step 2: Continuation (runs BEFORE other tasks)");
}

// Other code from third-party library
thirdPartyCode();  // Runs AFTER your continuation completes
```

#### Visual Timeline

**Without Prioritized Continuation:**
```
0ms  ├─ Your Task (Part 1) starts
5ms  ├─ Your Task yields
5ms  ├─ Other Task #1 starts
12ms ├─ Other Task #1 completes
12ms ├─ Other Task #2 starts
18ms ├─ Other Task #2 completes
18ms ├─ Your Task (Part 2) resumes  ← User sees delay!
25ms └─ Your Task completes
```

**With Prioritized Continuation:**
```
0ms  ├─ Your Task (Part 1) starts
5ms  ├─ Your Task yields
5ms  ├─ Your Task (Part 2) resumes immediately  ← Prioritized!
12ms ├─ Your Task completes
12ms ├─ Other Task #1 starts
19ms ├─ Other Task #1 completes
19ms ├─ Other Task #2 starts
25ms └─ Other Task #2 completes
```

#### Key Takeaway

- **Without continuation:** Your code's execution can be fragmented by third-party tasks, causing unpredictable performance
- **With continuation:** Your code runs in logical chunks without third-party interruption, providing more consistent behavior

This is why **`scheduler.yield()` is the recommended approach** over `setTimeout()` for task yielding.

## Best Practices & Conclusion

### Main Recommendations:

1. **Yield to the main thread for critical, user-facing tasks**
   - Prioritize UI updates and visual responses
   - Use yielding between user-critical and background work

2. **Use `scheduler.yield()` (with cross-browser fallback) to:**
   - Ergonomically yield with `async`/`await`
   - Get prioritized continuations
   - Avoid interruption from third-party tasks

3. **Do as little work as possible in your functions**
   - Minimize the amount of computation per task
   - Break complex operations into smaller logical units


