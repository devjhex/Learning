1️⃣ What is memory in the browser/JS context?
- Memory in JavaScript is the computer's RAM (Random Access Memory) allocated to store data during program execution. It includes:
  - **Heap:** Where objects, arrays, closures, and DOM nodes are stored (dynamic, variable size)
  - **Stack:** Where primitives (numbers, strings, booleans) and function calls are stored (fixed size, LIFO - Last In First Out)
  - **Call Stack:** Tracks function execution and local variables
- The browser manages this memory automatically through garbage collection, but developers must understand how to avoid leaks.

2️⃣ What kinds of data are stored in memory?
- numbers, strings, booleans) and function calls are stored
3️⃣ Who manages memory in JavaScript?
- **The JavaScript Engine** (V8 in Chrome, SpiderMonkey in Firefox, JavaScriptCore in Safari) manages memory automatically through:
  - **Automatic allocation:** When you create variables, objects, or functions, memory is allocated automatically
  - **Garbage Collection:** The engine automatically frees memory for objects that are no longer reachable
  - **You don't directly control memory:** No explicit `free()` or `delete` for memory (unlike C/C++)
  - **You influence it indirectly:** By removing references, cleaning up listeners, and breaking circular references, you help the GC do its job better

4️⃣ What is garbage collection?
- Garbage collection is the removal of garbage memory from memory that is memory that is not being used and this is done automatically in JS

5️⃣ When does memory get freed?
- Memory is freed automatically during **Garbage Collection cycles** when an object becomes **unreachable:**

  - **Unreachable = no references to it** from root objects (global scope, call stack, DOM tree)
  - **After a function returns:** Local variables on the stack are freed immediately
  - **When you break a reference:** Setting a variable to `null` or deleting a property makes an object eligible for GC
  - **Event listener removal:** Removing listeners breaks references and allows objects to be freed
  - **Garbage collection timing:** The engine decides when to run GC (usually during idle time or when memory pressure is high)
  - **Mark-and-Sweep algorithm:** Modern engines mark reachable objects, then sweep away unmarked (unreachable) ones
- **Note:** You don't control WHEN GC runs, but you control WHAT becomes unreachable by managing references