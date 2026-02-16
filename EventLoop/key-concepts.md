# Event loop
## Learn how the JavaScript event loop handles async code. Understand the call stack, task queue, microtasks, and why Promises always run before setTimeout().

## How does JavaScript handle multiple things at once when it can only do one thing at a time? Why does this code print in a surprising order?

```
console.log('Start');

setTimeout(() => console.log('Timeout'), 0);

Promise.resolve().then(() => console.log('Promise'));

console.log('End');

// Output:
// Start
// End
// Promise
// Timeout

```
- Even with a 0ms delay, Timeout prints last. The answer lies in the event loop. It’s JavaScript’s mechanism for handling asynchronous operations while remaining single-threaded.

## What is the event loop?
- The event loop is JavaScript’s mechanism for executing code, handling events, and managing asynchronous operations. It coordinates execution by checking callback queues when the call stack is empty, then pushing queued tasks to the stack for execution. This enables non-blocking behavior despite JavaScript being single-threaded ensuring that tasks are run in order without blocking the main thread unnecessarily.

## The problem Javascript is single threaded
- Javascript can only do one thing at a time. There’s one call stack, one thread of execution.

```
// JavaScript executes these ONE AT A TIME, in order
console.log('First');   // 1. This runs
console.log('Second');  // 2. Then this
console.log('Third');   // 3. Then this

```

## Why is this a problem?
- This is a problem beause imagine if every operation blocked the entire program  like the fetch API.

```
// If fetch() was synchronous (blocking)...
const data = fetch('https://api.example.com/data'); // Takes 2 seconds
console.log(data);
// NOTHING else can happen for 2 seconds!
// - No clicking buttons
// - No scrolling
// - No animations
// - Complete UI freeze!

```

## So the event loop is like a scheduler for JS.
What the event loop does
	•	Acts like a task scheduler
	•	Ensures that:
	•	JS code runs
	•	Input events are handled
	•	Microtasks and macrotasks execute in proper order
	•	rAF updates happen before paint
	•	All while keeping the UI smooth and responsive

## How?
- JavaScript solves this by delegating long-running tasks to the browser (or Node.js), which handles them in the background. Functions like setTimeout() don’t block

```
console.log('Start');

// This doesn't block! Browser handles the timer
setTimeout(() => {
  console.log('Timer done');
}, 2000);

console.log('End');

// Output:
// Start
// End
// Timer done (after 2 seconds)
```

## The Javascript runtime environment
### The components of the runtime environment
## The call Stack
 - The Call Stack is where JavaScript keeps track of what function is currently running. It’s a LIFO (Last In, First Out) structure, like a stack of plates.
 ```
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(4);
 ```

### Call Stack progression

```
1. [printSquare]
2. [square, printSquare]
3. [multiply, square, printSquare]
4. [square, printSquare]        // multiply returns
5. [printSquare]                 // square returns
6. [console.log, printSquare]
7. [printSquare]                 // console.log returns
8. []                            // printSquare returns

```
## The Heap
- The Heap is a large, mostly unstructured region of memory where objects, arrays, and functions are stored. When you create an object, it lives in the heap.

```

const user = { name: 'Alice' };  // Object stored in heap
const numbers = [1, 2, 3];       // Array stored in heap

```

## Web API's / C++ (NODE.JS)
- These are NOT part of JavaScript itself! They’re provided by the environment:

### Browser APIs:
1. setTimeout, setInterval
2. fetch, XMLHttpRequest
3. DOM events (click, scroll, etc.)
4. requestAnimationFrame
5. Geolocation, WebSockets, IndexedDB

### Node.js APIs:
1. File system operations
2. Network requests
3. Timers
4. Child processes

- These are handled by the browser/Node.js runtime outside of JavaScript execution, allowing JavaScript to remain non-blocking.

### Task Queue (Macrotask Queue)
- The Task Queue holds callbacks from:

1. setTimeout and setInterval
2. I/O operations
3. UI rendering tasks
4. Event handlers (click, keypress, etc.)
5. setImmediate (Node.js)

- Tasks are processed one at a time, with potential rendering between them.

### Micro task Queue
- The Microtask Queue holds high-priority callbacks from:

1. Promise.then(), .catch(), .finally()
NOTE:
- The promise constructor runs synchronously  but the others run synchronously:
```
console.log('1');

new Promise((resolve) => {
  console.log('2');  // Runs SYNCHRONOUSLY!
  resolve();
}).then(() => {
  console.log('3');  // Async (microtask)
});

console.log('4');

// Output: 1, 2, 4, 3
// Note: '2' prints before '4'!

```
2. queueMicrotask()
3. Muz1tationObserver
4. Code after await in async functions
​

### Event loop
- The Event Loop is the orchestrator. Its job is simple but crucial:
```
FOREVER:
  1. Execute all code in the Call Stack until empty
  2. Execute ALL microtasks (until microtask queue is empty)
  3. Render if needed (update the UI)
  4. Take ONE task from the task queue
  5. Go to step 1
```

- The key insight: Microtasks can starve the task queue! If microtasks keep adding more microtasks, tasks (and rendering) never get a chance to run.

## How the event loop runs Step by Step
1. Example : Basic SetTimeout
```
console.log('Start');

setTimeout(() => {
  console.log('Timeout');
}, 0);

console.log('End');

```

- Output: Start, End, Timeout

### Why ?

Let's trace it step by step

#### Execute console.log("Start");
- Call Stack: [console.log] → prints “Start” → stack empty
```
Call Stack: [console.log('Start')]
Web APIs: []
Task Queue: []
Output: "Start"

```

#### Execute setTimeout();
- setTimeout is called → registers timer with Web APIs → immediately returns
```
Call Stack: []
Web APIs: [Timer: 0ms → callback]
Task Queue: []

```

#### Timer completes (0ms);
- Browser’s timer finishes → callback moves to Task Queue
```
Call Stack: []
Web APIs: []
Task Queue: [callback]

```

#### Execute console.log('End');
- But wait! We’re still running the main script!
```
Call Stack: [console.log('End')]
Task Queue: [callback]
Output: "Start", "End"

```

#### Main script complete, Event Loop checks queues;
- Call stack is empty → Event Loop takes callback from Task Queue
```
Call Stack: [callback]
Task Queue: []
Output: "Start", "End", "Timeout"

```
NOTE:
- Key insight: Even with a 0ms delay, setTimeout callback NEVER runs immediately. It must wait for:
1. The current script to finish
2. All microtasks to complete
3. Its turn in the task queue


2. Example 2: Promises VS SetTimeout

```

console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

```
** Output: 1, 4, 3, 2 **

## Why does 3 come before 2?

#### Synchronous code runs first;
- console.log('1') → prints “1” 
- setTimeout → registers callback in Web APIs → callback goes to Task Queue
- Promise.resolve().then() → callback goes to Microtask Queue
- console.log('4') → prints “4”

```
Output so far: "1", "4"
Microtask Queue: [Promise callback]
Task Queue: [setTimeout callback]

```
#### Microtasks run before tasks;
- Call stack empty → Event Loop checks Microtask Queue first
- Promise callback runs → prints “3”
```
Output so far: "1", "4", "3"
Microtask Queue: []
Task Queue: [setTimeout callback]

```
#### Task Queue processed;
- Call stack empty → Event Loop checks Microtask Queue first
- Promise callback runs → prints “3”
```
Output so far: "1", "4", "3"
Microtask Queue: []
Task Queue: [setTimeout callback]

```
NOTE: 
- Call stack holds the currently running functions.
- Statements execute in order inside that function.
- Function pops off only after it returns, not after each statement.

3. Example 3: Nested Microtasks

```
console.log('Start');

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    Promise.resolve().then(() => console.log('Promise 2'));
  });

setTimeout(() => console.log('Timeout'), 0);

console.log('End');

```

- Output: Start, End, Promise 1, Promise 2, Timeout
- Even though the second promise is created AFTER setTimeout was registered, it still runs first because the entire microtask queue must be drained before any task runs!


## Tasks vs Microtasks: The Complete Picture
​
### What Creates Tasks (Macrotasks)?
Source	                  Description
- setTimeout(fn, delay)	  Runs fn after at least delay ms
- setInterval(fn, delay)	Runs fn repeatedly every ~delay ms
- I/O callbacks	          Network responses, file reads
- UI Events	              click, scroll, keydown, mousemove
- setImmediate(fn)	      Node.js only, runs after I/O
- MessageChannel	        postMessage callbacks


## What Creates Microtasks?
Source	                      Description
- Promise.then/catch/finally	When promise settles
- async/await	                Code after await
- queueMicrotask(fn)	        Explicitly queue a microtask
- MutationObserver	          When DOM changes


## The Event loop Algorithm simplified
```

// Pseudocode for the Event Loop (per HTML specification)
while (true) {
  // 1. Process ONE task from the task queue (if available)
  if (taskQueue.hasItems()) {
    const task = taskQueue.dequeue();
    execute(task);
  }
  
  // 2. Process ALL microtasks (until queue is empty)
  while (microtaskQueue.hasItems()) {
    const microtask = microtaskQueue.dequeue();
    execute(microtask);
    // New microtasks added during execution are also processed!
  }
  
  // 3. Render if needed (browser decides, typically ~60fps)
  if (shouldRender()) {
    // 3a. Run requestAnimationFrame callbacks
    runAnimationFrameCallbacks();
    // 3b. Perform style calculation, layout, and paint
    render();
  }
  
  // 4. Repeat (go back to step 1)
}

```
NOTE:
- Microtask Starvation: If microtasks keep adding more microtasks, the task queue (and rendering!) will never get a chance to run:
```
// DON'T DO THIS - infinite microtask loop!
function forever() {
  Promise.resolve().then(forever);
}
forever(); // Browser freezes!

```

## JavaScript Timers: setTimeout, setInterval, requestAnimationFrame
- Now that you understand the event loop, let’s dive deep into JavaScript’s timing functions.


### SetTimeout Delayed Execution 

```

// Syntax
const timerId = setTimeout(callback, delay, ...args);

// Cancel before it runs
clearTimeout(timerId);


```

Basic usage 

```

// Run after 2 seconds
setTimeout(() => {
  console.log('Hello after 2 seconds!');
}, 2000);

// Pass arguments to the callback
setTimeout((name, greeting) => {
  console.log(`${greeting}, ${name}!`);
}, 1000, 'Alice', 'Hello');
// Output after 1s: "Hello, Alice!"

```

## Cancelling a timeout

```

const timerId = setTimeout(() => {
  console.log('This will NOT run');
}, 5000);

// Cancel it before it fires
clearTimeout(timerId);

```
​
## The “Zero Delay” Myth
- setTimeout(fn, 0) does NOT run immediately!

```
console.log('A');
setTimeout(() => console.log('B'), 0);
console.log('C');

// Output: A, C, B (NOT A, B, C!)

```

- Even with 0ms delay, the callback must wait for:
1. Current script to complete
2. All microtasks to drain
3. Its turn in the task queue

## The Minimum Delay (4ms Rule)
After 5 nested timeouts, browsers enforce a minimum 4ms delay:

```

let start = Date.now();
let times = [];

setTimeout(function run() {
  times.push(Date.now() - start);
  if (times.length < 10) {
    setTimeout(run, 0);
  } else {
    console.log(times);
  }
}, 0);

// Typical output (varies by browser/system): [1, 1, 1, 1, 4, 9, 14, 19, 24, 29]
// First 4-5 are fast, then 4ms minimum kicks in


```
NOTE
- setTimeout delay is a MINIMUM, not a guarantee!

```
const start = Date.now();

setTimeout(() => {
  console.log(`Actual delay: ${Date.now() - start}ms`);
}, 100);

// Heavy computation blocks the event loop
for (let i = 0; i < 1000000000; i++) {}

// Output might be: "Actual delay: 2547ms" (NOT 100ms!)

```

If the call stack is busy, the timeout callback must wait.

## setInterval: Repeated Execution

```
// Syntax
const intervalId = setInterval(callback, delay, ...args);

// Stop the interval
clearInterval(intervalId);

```

## Basic Usage

```

let count = 0;

const intervalId = setInterval(() => {
  count++;
  console.log(`Count: ${count}`);
  
  if (count >= 5) {
    clearInterval(intervalId);
    console.log('Done!');
  }
}, 1000);

// Output every second: Count: 1, Count: 2, ... Count: 5, Done!



Time:     0ms    1000ms   2000ms   3000ms
          │       │        │        │
setInterval│───────│────────│────────│
          │  300ms │  300ms │  300ms │
          │callback│callback│callback│
          │       │        │        │
          
The 1000ms is between STARTS, not between END and START


```

## The setInterval Drift Problem
setInterval doesn’t account for callback execution time:

```
// Problem: If callback takes 300ms, and interval is 1000ms,
// actual time between START of callbacks is 1000ms,
// but time between END of one and START of next is only 700ms

setInterval(() => {
  // This takes 300ms to execute
  heavyComputation();
}, 1000);

```

## Solution: Nested setTimeout
For more precise timing, use nested setTimeout:

```

// Nested setTimeout guarantees delay BETWEEN executions
function preciseInterval(callback, delay) {
  function tick() {
    callback();
    setTimeout(tick, delay);  // Schedule next AFTER current completes
  }
  setTimeout(tick, delay);
}

// Now there's exactly `delay` ms between the END of one
// callback and the START of the next

Time:     0ms    1300ms   2600ms   3900ms
          │       │        │        │
Nested    │───────│────────│────────│
setTimeout│  300ms│  300ms │  300ms │
          │   +   │   +    │   +    │
          │ 1000ms│ 1000ms │ 1000ms │
          │ delay │ delay  │ delay  │

```

NOTE:
When to use which:
setInterval: For simple UI updates that don’t depend on previous execution
Nested setTimeout: For sequential operations, API polling, or when timing precision matters

## requestAnimationFrame: Smooth Animations
requestAnimationFrame (rAF) is designed specifically for animations. It syncs with the browser’s refresh rate (usually 60fps = ~16.67ms per frame).

```
// Syntax
const rafId = requestAnimationFrame(callback);

// Cancel
cancelAnimationFrame(rafId);

```

### Basic animation loop
```
function animate(timestamp) {
  // timestamp = time since page load in ms
  
  // Update animation state
  element.style.left = (timestamp / 10) + 'px';
  
  // Request next frame and to continue running the code and the function in the next and next frames.
  requestAnimationFrame(animate);
}

// Start the animation for  the current rAF phase in the current frame
requestAnimationFrame(animate);

```


### Why requestAnimationFrame is Better for Animations
Feature	             setTimeout/setInterval	            requestAnimationFrame
Sync with display	            No	                      Yes (matches refresh rate)
Battery efficient	            No	                      Yes (pauses in background tabs)
Smooth animations	        Can be janky	                Optimized by browser
Timing accuracy	          Can drift	                    Consistent frame timing
CPU usage	                Runs even if tab hidden	      Pauses when tab hidden

### Example: Animating with rAF

```

const box = document.getElementById('box');
let position = 0;
let lastTime = null;

function animate(currentTime) {
  // Handle first frame (no previous time yet)
  if (lastTime === null) {
    lastTime = currentTime;
    requestAnimationFrame(animate);
    return;
  }
  
  // Calculate time since last frame
  const deltaTime = currentTime - lastTime;
  lastTime = currentTime;
  
  // Move 100 pixels per second, regardless of frame rate
  const speed = 100; // pixels per second
  position += speed * (deltaTime / 1000);
  
  box.style.transform = `translateX(${position}px)`;
  
  // Stop at 500px
  if (position < 500) {
    requestAnimationFrame(animate);
  }
}

requestAnimationFrame(animate);

```

### When requestAnimationFrame Runs
```

One Event Loop Iteration:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Run task from Task Queue                                     │
├─────────────────────────────────────────────────────────────────┤
│ 2. Run ALL microtasks                                           │
├─────────────────────────────────────────────────────────────────┤
│ 3. If time to render:                                           │
│    a. Run requestAnimationFrame callbacks  ← HERE!              │
│    b. Render/paint the screen                                   │
├─────────────────────────────────────────────────────────────────┤
│ 4. If idle time remains before next frame:                      │
│    Run requestIdleCallback callbacks (non-essential work)       │
└─────────────────────────────────────────────────────────────────┘

```

## Blocking the Event Loop
​
What Happens When You Block?
When synchronous code runs for a long time, EVERYTHING stops:

```

// This freezes the entire page!
button.addEventListener('click', () => {
  // Heavy synchronous work
  for (let i = 0; i < 10000000000; i++) {
    // ... computation
  }
});

```

## Consequences:
- UI freezes (can’t click, scroll, or type)
- Animations stop
- setTimeout/setInterval callbacks delayed
- Promises can’t resolve
- Page becomes unresponsive


# Solutions
## Web Workers
​- Move heavy computation to a separate thread using Web Workers:
```

// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: largeArray });

worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// worker.js
self.onmessage = (event) => {
  const result = heavyComputation(event.data);
  self.postMessage(result);
};

```

## Chunking with SetTimout
Break work into smaller chunks:
```

function processInChunks(items, process, chunkSize = 100) {
  let index = 0;
  
  function doChunk() {
    const end = Math.min(index + chunkSize, items.length);
    
    for (; index < end; index++) {
      process(items[index]);
    }
    
    if (index < items.length) {
      setTimeout(doChunk, 0); // Yield to event loop
    }
  }
  
  doChunk();
}

// Now UI stays responsive between chunks
processInChunks(hugeArray, item => compute(item));
```

## requestIdleCallback
Run code during browser idle time with requestIdleCallback:

```
function doNonCriticalWork(deadline) {
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    const task = tasks.shift();
    task();
  }
  
  if (tasks.length > 0) {
    requestIdleCallback(doNonCriticalWork);
  }
}

requestIdleCallback(doNonCriticalWork);

```

## Rendering and the Event Loop. 
### Where Does Rendering Fit?

The browser tries to render at 60fps (every ~16.67ms). Rendering happens between tasks, after microtasks:

```

┌─────────────────────────────────────────────────────┐
│                 One Frame (~16.67ms)                │
├─────────────────────────────────────────────────────┤
│  1. Task (from Task Queue)                          │
│  2. All Microtasks                                  │
│  3. requestAnimationFrame callbacks                 │
│  4. Style calculation                               │
│  5. Layout                                          │
│  6. Paint                                           │
│  7. Composite                                       │
└─────────────────────────────────────────────────────┘

```


## Why 60fps Matters
FPS	        Frame Time	      User Experience
60	        16.67ms	          Smooth, responsive
30	        33.33ms	          Noticeable lag
15	        66.67ms	          Very choppy
< 10	>     100ms	            Unusable

If your JavaScript takes longer than ~16ms, you’ll miss frames and the UI will feel janky.
​

## Using requestAnimationFrame for Visual Updates
Use rAF to avoid layout thrashing (reading and writing DOM in a way that forces multiple reflows):

```

// Bad: Read-write-read pattern forces multiple layouts
console.log(element.offsetWidth);     // Read (forces layout)
element.style.width = '100px';        // Write
console.log(element.offsetHeight);    // Read (forces layout AGAIN!)
element.style.height = '200px';       // Write

// Good: Batch reads together, then defer writes to rAF
const width = element.offsetWidth;    // Read
const height = element.offsetHeight;  // Read (same layout calculation)

requestAnimationFrame(() => {
  // Writes happen right before next paint
  element.style.width = width + 100 + 'px';
  element.style.height = height + 100 + 'px';
});

```

# Key Takeaways
The key things to remember:
JavaScript is single-threaded — only one thing runs at a time on the call stack
The Event Loop enables async — it coordinates between the call stack and callback queues
Web APIs run in separate threads — timers, network requests, and events are handled by the browser
Microtasks > Tasks — Promise callbacks ALWAYS run before setTimeout callbacks
setTimeout delay is a minimum — actual timing depends on call stack and queue state
setInterval can drift — use nested setTimeout for precise timing
requestAnimationFrame for animations — syncs with browser refresh rate, pauses in background
Never block the main thread — long sync operations freeze the entire UI
Microtasks can starve tasks — infinite microtask loops prevent rendering
The Event Loop isn’t JavaScript — it’s part of the runtime environment (browser/Node.js)




