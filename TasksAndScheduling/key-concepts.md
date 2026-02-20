# Key concepts

## Task

**Definition:** A discrete unit of work the browser executes on the main thread.

## Scheduling

**Definition:** The process of controlling when and in what order tasks execute on the main thread, and yielding execution to allow the browser to process higher-priority work (user interactions, rendering).

## Task Queue

**Definition:** The queue where tasks wait to be executed on the main thread. When you yield, your task gets added to the queue. With `scheduler.yield()`, your continuation is prioritized; with `setTimeout()`, it goes to the end.


## Yielding

**Definition:** Intentionally pausing execution to give the main thread an opportunity to process higher-priority work before resuming.