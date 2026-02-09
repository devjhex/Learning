# Say your mouse generates 10 raw events in a frame, but the main thread is free and JS runs immediately.
	•	Browser sees the first raw event → calls your JS handler → dispatches 1 event
	•	All remaining raw events either happen after this frame, or JS handles them individually in the next frame

# What does that mean?

Step 1: What’s happening physically
	•	Your mouse moves → OS sends 10 raw signals in 16.6ms (1 frame).
	•	Main thread is free, i.e., nothing blocking JS.

⸻

Step 2: How the browser sees them
	1.	First raw event (E1) arrives.
	•	Browser immediately dispatches it to JS.
	•	dispatch = JS handler runs.
	•	getCoalescedEvents().length = 1
	2.	Remaining raw events (E2…E10) arrive after E1’s dispatch:
	•	JS is still executing, but the frame is not over
	•	Browser decides: “No need to merge, the main thread can handle these individually”
	•	Each event may get dispatched in its own next microtask / next frame tick

⸻

Step 3: Why you see “1” in light pages
	•	Because the first dispatch happens immediately, it contains only E1.
	•	The remaining raw events may:
	•	Not have arrived yet
	•	Or get dispatched separately in subsequent frames
	•	So if you log getCoalescedEvents() during that first dispatch, it will show 1.

✅ No merging was needed because JS could keep up.

⸻

Step 4: Contrast with heavy pages
	•	Same 10 raw events
	•	Main thread is busy, cannot dispatch immediately
	•	Browser buffers all 10 raw events
	•	Once the main thread is free (still within the same frame), one dispatch happens, containing all 10 events
	•	getCoalescedEvents().length = 10

✅ Only one JS call, but with multiple raw events inside it.

⸻

Step 5: Why the “light page” can still have 10 events in total
	•	They’re just dispatched one by one, not merged.
	•	You might see 10 separate JS calls, but each contains 1 raw event.

So:

Page Type	Raw Events	Dispatches	getCoalescedEvents()
Light	10	10	1 per dispatch
Heavy	10	1	10


⸻

🔑 Takeaway

Dispatch = JS handler call
Coalescing only happens when the browser has multiple raw events before it can dispatch
Light pages → JS keeps up → dispatches immediately → little/no coalescing
Heavy pages → JS blocked → multiple raw events merged → one dispatch

⸻

## More Details about this topic
1️⃣ What getCoalescedEvents() actually is
	•	Returns an array of all the raw input events that were merged into this single JS dispatch.
	•	Each item = one “raw” event the hardware/OS generated.


2️⃣ Light page (JS can keep up)
	•	Only 1 raw event arrived before the dispatch
	•	getCoalescedEvents() → [E1] → length = 1
	•	✅ Nothing was merged because the main thread could handle events as they arrived

3️⃣ Heavy page (main thread busy)
	•	10 raw events arrived before the dispatch
	•	Browser merges them into 1 dispatch
	•	getCoalescedEvents() → [E1, E2, …, E10] → length = 10
	•	✅ Shows all the inputs the browser combined into a single JS call

4️⃣ Key mental shortcut
Length of getCoalescedEvents() = how many raw inputs were merged into this one dispatch

	•	1 → no coalescing this frame (JS kept up)
	•	>1 → coalescing happened (multiple raw inputs merged into 1 dispatch)
	•	Time window = roughly one frame (~16.6ms at 60Hz)


So you can now read this as:
	•	“What raw inputs did my JS actually see in this frame?”

And that’s exactly why this method exists — you don’t have to handle 50 separate events per frame manually.

## Browser Input & Coalescing — Full Summary

Step 0: Physical Input
	•	User moves mouse / finger → OS generates raw input events (E1, E2, …).
	•	Happens independently of JS or browser.

⸻

Step 1: Browser Receives Events
	•	Browser collects raw events.
	•	At this point: no JS has run yet.
	•	Events are just queued internally.

⸻

Step 2: Dispatch to JS
	•	Dispatch = calling your JS event listeners.
	•	Browser decides when to dispatch:

Rule: Continuous high-frequency events
	•	mousemove, pointermove, wheel, touchmove

Light page / main thread free:
	•	Browser can dispatch each raw input immediately.
	•	getCoalescedEvents().length = 1
	•	JS sees every event separately → no merging needed.

Heavy page / main thread busy:
	•	Browser buffers multiple raw inputs.
	•	One dispatch contains multiple raw events.
	•	getCoalescedEvents().length > 1
	•	JS sees all events in one call, coalesced.

⸻

Step 3: requestAnimationFrame (rAF)
	•	rAF runs once per frame (~16.6ms at 60Hz).
	•	Purpose: synchronize JS visual updates with browser’s rendering pipeline.

Order inside a frame:

1. Dispatch input events (JS handlers)
2. requestAnimationFrame callbacks
3. Style / Layout / Paint
4. Composite → GPU → Screen

	•	Continuous events are always dispatched right before rAF.
	•	Ensures visuals match the latest input.

⸻

Step 4: Coalescing
	•	Happens automatically for continuous events.
	•	Only one JS dispatch per frame.
	•	getCoalescedEvents() shows all raw inputs merged.
	•	Prevents:
	•	Multiple layouts / paints per frame
	•	Jank
	•	Wasted CPU

⸻

Step 5: Key differences: light vs heavy page

Feature	Light page	Heavy page
Main thread load	Free	Busy
Raw events per frame	1-2	10+
Dispatch count	1 per raw event	1 per frame
getCoalescedEvents().length	1	10+
Visual effect	Immediate, no merge	Coalesced, still smooth


⸻

Step 6: Mental picture
	•	Conveyor belt analogy:

Light page:

[raw E1] → dispatch → JS → visuals
[raw E2] → next dispatch → JS → visuals

Heavy page:

[raw E1, E2, …, E10] → ONE dispatch → JS sees all → visuals

	•	Dispatch = JS sees event
	•	Coalescing = multiple raw events merged into one dispatch

⸻

Step 7: Rules you can use as a dev
	1.	Never assume 1 raw event = 1 JS call
	2.	Always check getCoalescedEvents() if precise motion matters
	3.	Use rAF to read latest input and update visuals once per frame
	4.	Continuous events are always coalesced by default, even when page is light
	5.	Passive listeners let scroll bypass JS for smooth performance

⸻

✅ One-sentence anchor:

Raw inputs arrive continuously, dispatch is when JS runs, coalescing merges multiple raw inputs into a single dispatch per frame, and rAF aligns JS updates with the browser’s rendering to ensure smooth visuals.

⸻

If you want, I can make a diagram that shows a frame timeline for light vs heavy pages, with raw events, dispatch, coalescing, rAF, layout, paint, and composite — it usually locks this entire concept visually.

Do you want me to make that diagram?