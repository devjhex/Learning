What confused me at first?
	•	What clicked?
	•	What mental model changed?
	•	What surprised me?

	What clicked?
	### What hapenens when there is a user gesture like a touch etc?
	- Input events are receieved by the browser process which forwards co-ordinate based data to the appropriate renderer process wherer DOM hit testing  and JS handling occur (unless the compositor can handle it directly)

	NOTE: 
	It might be that most of the timethe browser paints the entire document into a single large composited layer and the rest of the layers that are created only as performance optimizations.

	### What is non-fast scrollable region?
	Since running JS is the main thread's job, the compositor marks off a region that has event handlers attached to it as a non-fast scrollable region. By having this information, the compositor thread can make sure to send input event to the main thread if the event occurs in that region.

	A non-scrollabe region is a part of the page where the browser must wait for javascript on the main thread  before it is allowed to scroll because the scroll might be cancelled. So the compositor cannot scroll on its own
	
	### Be careful when writing your event handlers
	The existence of event handlers is like asking this question "Can the compositor scroll without asking?"

	For example :
	1. BAD: This is bad for scrolling:
	```

	document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
      
    }
	});

	```
	Why.....
	a. The region has become non-fast scrollable region
	b. Main thread must be consulted
	c. Potential frame drops

	2. BAD : This is bad for scrolling: 
	```
	document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
        event.preventDefault();
    }
	});
	```
	Why .....
	a. The entire page is marked as non-fast scrollable
	b. Main thread must be consulted on every event since its on a large scale now
	c. Potential frame drops
	d. The smooth scrolling ability of the compositor is defeated.



	3. GOOD: This is good for scrolling:
	```

	document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
      event.preventDefault()
    }
	}, {passive:true});

	```
	Why.....
	You are now telling the browser "I promise i will not block scrolling"
	a. The compositor scrolls immediately
	b. No main thread wait
	c. Smooth scrolling

	### Cancelable Events.
	- JS can block the browser's default behavior (eg. scrolling, link navigation)
	- Browser must wait  for JS (this could cause scroll jank if the main thread is busy)
	- Example normal touchmove or click events

	Passive listeners
	- JS cannot block teh default behavior
	- Browsers can scroll immediately on the compositor -- smooth scrolling (60fps)
	- Use for : general scroll tracking, analytics, or any listener that doesn't need to cancel

	```
	element.addEventListener("touchmove", handler, {passive:true})
	```
	Checking Event.cancelable
	- Only needed if you plan to call preventDefault()
	- Prevents warnings  if the event is not cancelable (passive listeners)

	```
	element.addEventListener('touchmove', e=>{
		if(e.cancelable){
			e.preventDefault();
		}
	})
	```

	What happens if you don't check?
	- if the event is cancelable - works fine
	- if the event is not cancelable - call is ignored browser may warn

	This is defensive programming.

	Alternatively, you may use CSS rule like touch-action to completely eliminate the event handler.

	```
	#area {
  		touch-action: pan-x;
	}
	```

	### Finding the event target
	- When the compositor thread sends an input event to the main thread, the first thing to run is a hit test to find the event target. Hit test uses paint records data that was generated in the rendering process to find out what is underneath the point coordinates in which the event occurred.

	### Minimizing event dispatches on the main thread

	- If a continuous event like touchmove was sent to the main thread 120 times a second, then it might trigger excessive amount of hit tests and JavaScript execution compared to how slow the screen can refresh.

	- To minimize excessive calls to the main thread, Chrome coalesces continuous events (such as wheel, mousewheel, mousemove, pointermove, touchmove ) and delays dispatching until right before the next requestAnimationFrame.

	- Any discrete events like keydown, keyup, mouseup, mousedown, touchstart, and touchend are dispatched immediately.

	### Use getCoalescedEvents to get intra-frame Events
	- IF you want to get the small events that happened in a curve since they are coalesced by the main thread you can use the getCoalescedEvents() so that you have access to these events. This is for those creating like drawing applicaitons so as not to lose anything (any of the events).

	```
	window.addEventListener('pointermove', event => {
    const events = event.getCoalescedEvents();
    for (let event of events) {
        const x = event.pageX;
        const y = event.pageY;
        // draw a line using x and y coordinates.
    }
	});
```


What mental model changed?

Perfect — let’s summarize everything about input, dispatch, coalescing, rAF, and frames into one crystal-clear picture so you can lock it in forever. I’ll go step by step with order, differences, and meaning.

⸻

Browser Input & Coalescing — Full Summary

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



