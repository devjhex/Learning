# What is the event loop and how does it work?

- The event loop is a JS super power which helps JS be able to do many things at the same time even though by default it can only be able to do one thing at the same time.


# How

- The event loop does it by having 3 rooms, the microtask queue, the macrotask queue(task queue) and the call stack.

- Since it has to make sure everything is away very fast and at the same time, it makes sure what all tasks that are supposed to go to the microtask room are put there, the tasks that are supposed to be put in the macrotask room, are put thrown there and the 3rd room which is the callstack is also where everything gets counted.

- So all the fast stuff go the callstack room and are counted fast, then if there are any things in the microtask room, then all of them are counted next, then finally one thing from the task room is counted as well and that is repeated again and again.

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