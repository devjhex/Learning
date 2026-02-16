# The reflections of the Event loop

# What is the event loop?
- The event loop is JS's mechanism for executing code, handling asynchronous operations and events. 
- It does this by checking callback queues when the call stack is empty and pushing more tasks from the queues to the stack for execution.
- It does this not to block JS since it is single threaded.


# Is JS single threaded? And why is this a problem?
- Yes JS is single threaded and this means it executes one thing at a time.
```
// JavaScript executes these ONE AT A TIME, in order
console.log('First');   // 1. This runs
console.log('Second');  // 2. Then this
console.log('Third');   // 3. Then this
```

## This is a problem because it means if anything blocks the code and it takes a long time everything else would have to freeze and that means lagging or freezing of other parts of the browser.

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

Imagine if every operation blocked the entire program.


# Solution to that: Asynchronous JS
- This is the solution and this is what happens, JS gets the long tasks and delegates them to the browser or Node.js and that means the long tasks are off JS and are handled by the browser not JS. so functions like setTimeout dont block.
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

# The JS runtime environment

```

┌─────────────────────────────────────────────────────────────────────────┐
│                        JAVASCRIPT RUNTIME                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      JAVASCRIPT ENGINE (V8, SpiderMonkey, etc.) │   │
│  │  ┌───────────────────────┐    ┌───────────────────────────┐     │   │
│  │  │      CALL STACK       │    │          HEAP             │     │   │
│  │  │                       │    │                           │     │   │
│  │  │  ┌─────────────────┐  │    │   { objects stored here } │     │   │
│  │  │  │ processData()   │  │    │   [ arrays stored here ]  │     │   │
│  │  │  ├─────────────────┤  │    │   function references     │     │   │
│  │  │  │ fetchUser()     │  │    │                           │     │   │
│  │  │  ├─────────────────┤  │    │                           │     │   │
│  │  │  │ main()          │  │    │                           │     │   │
│  │  │  └─────────────────┘  │    └───────────────────────────┘     │   │
│  │  └───────────────────────┘                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    BROWSER / NODE.js APIs                        │   │
│  │                                                                  │   │
│  │   setTimeout()    setInterval()    fetch()    DOM events         │   │
│  │   requestAnimationFrame()    IndexedDB    WebSockets             │   │
│  │                                                                  │   │
│  │   (These are handled outside of JavaScript execution!)           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    │                                    │
│                                    │ callbacks                          │
│                                    ▼                                    │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  MICROTASK QUEUE                    TASK QUEUE (Macrotask)       │  │
│  │  ┌────────────────────────┐        ┌─────────────────────────┐   │  │
│  │  │ Promise.then()         │        │ setTimeout callback     │   │  │
│  │  │ queueMicrotask()       │        │ setInterval callback    │   │  │
│  │  │ MutationObserver       │        │ I/O callbacks           │   │  │
│  │  │ async/await (after)    │        │ UI event handlers       │   │  │
│  │  └────────────────────────┘        │ Event handlers          │   │  │
│  │         ▲                          └─────────────────────────┘   │  │
│  │         │ HIGHER PRIORITY                    ▲                   │  │
│  └─────────┼────────────────────────────────────┼───────────────────┘  │
│            │                                    │                       │
│            └──────────┬─────────────────────────┘                       │
│                       │                                                 │
│              ┌────────┴────────┐                                        │
│              │   EVENT LOOP    │                                        │
│              │                 │                                        │
│              │  "Is the call   │                                        │
│              │   stack empty?" ├──────────► Push next callback          │
│              │                 │            to call stack               │
│              └─────────────────┘                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

# The Event loop in simple terms
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
For the async/await everything before the await keyword is synchronous until we reach the await keyword and we do all that is asynchronous hence its put in the microtask queue.
