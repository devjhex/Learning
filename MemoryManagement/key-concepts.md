# Key concepts

## Memory management
Memory management is the process of allocating, using, and releasing memory in your application.

NOTE:
In JS the concept of memory management is automatic (garbage collection).

- **Allocation:** Memory is reserved for variables, objects, arrays, closures, DOM nodes, etc. Happens when you create new values or references.

- **Reachability:** As long as a value is reachable (referenced from root objects like `window`, `global`, or the call stack), it won't be collected.

- **Garbage Collection:** The JS engine automatically frees memory for unreachable objects. Most engines use mark-and-sweep algorithms.

- **Leaks:** Memory leaks happen when objects are no longer needed but are still referenced (e.g., forgotten event listeners, closures holding onto DOM nodes, global variables).

- **Manual Release:** You can't explicitly free memory, but you can null references, remove event listeners, and break circular references to help GC. Lovely

## What You Need to Know

- **Avoid global variables** and long-lived references unless necessary.

- **Clean up event listeners** and intervals/timeouts when elements are removed.

- **Be careful with closures** that capture large objects or DOM nodes.

- **Detach DOM nodes** before deleting or replacing them.

- **Use tools:** Chrome DevTools (Memory tab, heap snapshots, allocation instrumentation) to find and fix leaks.

- **Watch for detached DOM trees** and uncollected timers.

- **Large arrays/objects:** Set to `null` or reassign to release memory if no longer needed.


What to DO:
- JS memory is managed for you, but leaks are still possible.
- Know what keeps objects alive (roots, references).
- Clean up after yourself (listeners, timers, DOM nodes).
- Use DevTools to monitor and debug memory usage.