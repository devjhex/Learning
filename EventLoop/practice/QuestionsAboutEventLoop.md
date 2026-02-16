Classic Interview Questions
​
# Question 1: Basic Output Order
```
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```
# Answer
1, 4, 3, 2.

# Explanation:
console.log('1') — synchronous, runs immediately → “1”
setTimeout — callback goes to Task Queue
Promise.then — callback goes to Microtask Queue
console.log('4') — synchronous, runs immediately → “4”
Call stack empty → drain Microtask Queue → “3”
Microtask queue empty → process Task Queue → “2”

​
# Question 2: Nested Promises and Timeouts
```
setTimeout(() => console.log('timeout 1'), 0);

Promise.resolve().then(() => {
  console.log('promise 1');
  Promise.resolve().then(() => console.log('promise 2'));
});

setTimeout(() => console.log('timeout 2'), 0);

console.log('sync');
```
# Answer
'sync', 'promise 1', 'promise 2', 'timeout 1' , 'timeout 2'

# Explanation:
First setTimeout → callback to Task Queue
Promise.then → callback to Microtask Queue
Second setTimeout → callback to Task Queue
console.log('sync') → runs immediately → “sync”
Drain Microtask Queue:
Run first promise callback → “promise 1”
This adds another promise to Microtask Queue
Continue draining → “promise 2”
Microtask Queue empty, process Task Queue:
First timeout → “timeout 1”
Second timeout → “timeout 2”

​
Question 3: async/await Ordering
```
async function foo() {
  console.log('foo start');
  await Promise.resolve();
  console.log('foo end');
}

console.log('script start');
foo();
console.log('script end');
```

# Answer
'script start', 'foo start', 'script end', 'foo end'

# Explanation:
console.log('script start') → “script start”
Call foo():
console.log('foo start') → “foo start”
await Promise.resolve() — pauses foo, schedules continuation as microtask
foo() returns (suspended at await)
console.log('script end') → “script end”
Call stack empty → drain Microtask Queue → resume foo
console.log('foo end') → “foo end”
Key insight: await splits the function. Code before await runs synchronously. Code after await runs as a microtask.
​
# Question 4: setTimeout in a Loop
```
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
Answer
3, 3, 3

# Explanation:
var is function-scoped, so there’s only ONE i variable
The loop runs synchronously: i=0, i=1, i=2, i=3 (loop ends)
THEN the callbacks run, and they all see i = 3

## Fix with let:
```
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

## Fix with closure (IIFE):
```
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 0);
  })(i);
}
// Output: 0, 1, 2
```

## Fix with setTimeout's third parameter
```
for (var i = 0; i < 3; i++) {
  setTimeout((j) => console.log(j), 0, i);
}
// Output: 0, 1, 2
```


​
# Question 5: What’s Wrong Here?
```
const start = Date.now();
setTimeout(() => {
  console.log(`Elapsed: ${Date.now() - start}ms`);
}, 1000);

// Simulate heavy computation
let sum = 0;
for (let i = 0; i < 1000000000; i++) {
  sum += i;
}
console.log('Heavy work done');
```
Answer
- The synchronous code is very heavy which blocks the event loop.

## Problem: The timeout will NOT fire after 1000ms!
The heavy for loop blocks the call stack. Even though the timer finishes after 1000ms, the callback cannot run until the call stack is empty.
Typical output:
```
Heavy work done
Elapsed: 3245ms  // Much longer than 1000ms!
```
Lesson: Never do heavy synchronous work on the main thread. Use:
Web Workers for CPU-intensive tasks
Break work into chunks with setTimeout
Use requestIdleCallback for non-critical work

​
# Question 6: Microtask Starvation
```
function scheduleMicrotask() {
  Promise.resolve().then(() => {
    console.log('microtask');
    scheduleMicrotask();
  });
}

setTimeout(() => console.log('timeout'), 0);
scheduleMicrotask();
```

# Answer
Output: microtask, microtask, microtask, … (forever!)
The timeout callback NEVER runs!
Explanation:
Each microtask schedules another microtask
The Event Loop drains the entire microtask queue before moving to tasks
The microtask queue is never empty
The timeout callback starves
This is a browser freeze! The page becomes unresponsive because rendering also waits for the microtask queue to drain.


​
Common Misconceptions
## Misconception 1: 'setTimeout(fn, 0) runs immediately'
Wrong! Even with 0ms delay, the callback goes to the Task Queue and must wait for:
Current script to complete
All microtasks to drain
Its turn in the queue

```
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('sync');

// Output: sync, promise, timeout (NOT sync, timeout, promise)
```


# Misconception 2: 'setTimeout delay is guaranteed'
Wrong! The delay is a MINIMUM wait time, not a guarantee.
If the call stack is busy or the Task Queue has items ahead, the actual delay will be longer.

```
setTimeout(() => console.log('A'), 100);
setTimeout(() => console.log('B'), 100);

// Heavy work takes 500ms
for (let i = 0; i < 1e9; i++) {}

// Both A and B fire at ~500ms, not 100ms
```


# Misconception 3: 'JavaScript is asynchronous'

Partially wrong! JavaScript itself is single-threaded and synchronous.
The asynchronous behavior comes from:
The runtime environment (browser/Node.js)
Web APIs that run in separate threads
The Event Loop that coordinates callbacks
JavaScript code runs synchronously, one line at a time. The magic is that it can delegate work to the environment.

# Misconception 4: 'The Event Loop is part of JavaScript'
Wrong! The Event Loop is NOT defined in the ECMAScript specification.
It’s defined in the HTML specification (for browsers) and implemented by the runtime environment. Different environments (browsers, Node.js, Deno) have different implementations.

# Misconception 5: 'setInterval is accurate'

Wrong! setInterval can drift, skip callbacks, or have inconsistent timing.
If a callback takes longer than the interval, callbacks queue up
Browsers may throttle timers in background tabs
Timer precision is limited (especially on mobile)
For precise timing, use nested setTimeout or requestAnimationFrame.


Test Your Knowledge
# Question 1: What is the Event Loop's main job?
- The Event loop's main job is to make sure it handles events and operations in the most synchronouse way possible with rules as to what comes first and in what order just to make sure the main thread is not blocked.

# Question 2: Why do Promises run before setTimeout?
- Promises run before setTimeout because they are micro tasks and those are of high priority by the event loop.


# Question 3: What's the output of this code?
```

setTimeout(() => console.log('A'), 0);
Promise.resolve().then(() => console.log('B'));
Promise.resolve().then(() => {
  console.log('C');
  setTimeout(() => console.log('D'), 0);
});
console.log('E');

```

# Answer
 E, B, C, A, D
E — synchronous
B — first microtask
C — second microtask (also schedules timeout D)
A — first timeout
D — second timeout (scheduled during microtask C)



# Question 4: When should you use requestAnimationFrame?
# Answer
## Use requestAnimationFrame for:
Visual animations
DOM updates that need to be smooth
Anything that should sync with the browser’s refresh rate

## Don’t use it for:
Non-visual delayed execution (use setTimeout)
Repeated non-visual tasks (use setInterval or setTimeout)
Heavy computation (use Web Workers)

# Question 5: What's wrong with this code?

```
setInterval(async () => {
  const response = await fetch('/api/data');
  const data = await response.json();
  updateUI(data);
}, 1000);
```
# Answer: 
- If the fetch takes longer than 1 second, multiple requests will be in flight simultaneously, potentially causing race conditions and overwhelming the server.

# Better Approach
```
async function poll() {
  const response = await fetch('/api/data');
  const data = await response.json();
  updateUI(data);
  setTimeout(poll, 1000); // Schedule next AFTER completion
}
poll();
<!-- Now:
	•	Request finishes
	•	Then 1 second later → next request
	•	No overlap
	•	No drift explosion
	•	No stacking -->
```
Why this happens (event loop explanation)

setInterval places a macrotask into the task queue every 1000ms.

The async function:
	•	runs
	•	pauses at await
	•	returns control to the event loop

But setInterval doesn’t track that promise.

It just keeps scheduling new macrotasks.


Question 6: How can you yield to the Event Loop in a long-running task?
```
// 1. setTimeout (schedules a task)
await new Promise(resolve => setTimeout(resolve, 0));

// 2. queueMicrotask (schedules a microtask)
await new Promise(resolve => queueMicrotask(resolve));

// 3. requestAnimationFrame (syncs with rendering)
await new Promise(resolve => requestAnimationFrame(resolve));

// 4. requestIdleCallback (runs during idle time)
await new Promise(resolve => requestIdleCallback(resolve));
```
- Each has different timing and use cases. setTimeout is most common for yielding.

