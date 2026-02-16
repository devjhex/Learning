# KEY CONCEPTS

## What is a task?
- A task is a piece of work that the browser does.

## Examples of work
- rendering
- parsing
- running JS
- etc

BuUUUUT JS is the largest source of tasks and it does impact performance in two main ways:
- During Startup
- Tasks are queued when JS has to respond to interactions (event handlers), js driven animations, and background activity.


## What is the main thread?
- This is where most most tasks are run in the browser and alomst where all JS is executed in the browser

### User experience impact
- UI feels laggy
- page may feel brokena and not nice if the main thread is blocked for a long time

**Visual Effect:** When a user interacts with a page and an event handler is queued during a long task, that handler must wait. If the task is broken into smaller chunks, the event handler can begin much sooner, making the interaction feel instant rather than laggy.

## Strategies
### 1. Manually Defer Code Execution with `setTimeout()`
### 2. Breaking Up Functions (Not Effective Alone)
hile breaking work into smaller functions is good software architecture, **JavaScript runs all called functions as a single task** because they execute within the parent function's context.

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


Mental model
	•	Promise.resolve() → “resume very soon, after current task.”
	•	setTimeout(0) → “resume later, after the browser does whatever else is queued.”
	•	scheduler.yield() → “pause me, let the browser breathe, then continue as soon as the browser is ready, before other low-priority stuff.”

![table of the difference of how they move stuff](images/Screenshot%202026-02-16%20at%2014.53.38.png)
