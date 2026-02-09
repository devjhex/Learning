# Non-fast scrollable region

Absolutely — let’s make this super visual, so you can actually “see” what a non-fast scrollable region is and why it matters for the compositor.

⸻

Non-Fast Scrollable Region — What It Is

Quick definition in one sentence:

A non-fast scrollable region is a part of the page that cannot be scrolled independently by the compositor thread alone, so scrolling it requires asking the main thread.

	•	Contrast: a “fast scrollable region” (like a normal div with overflow: scroll that the compositor can move independently) can scroll without touching JS.

⸻

Visual Example

Imagine a webpage with two areas:

---------------------------------------
|           HEADER (fixed)             |
---------------------------------------
|   SCROLLABLE DIV (fast scrollable)  |
|   ┌─────────────────────────────┐   |
|   │   Content inside div        │   |
|   │   Lots of text/images       │   |
|   └─────────────────────────────┘   |
|                                     |
|   PAGE BODY CONTENT (non-fast)      |
|   Lots of text/images              |
---------------------------------------
|           FOOTER                     |
---------------------------------------

	•	SCROLLABLE DIV → compositor can scroll this smoothly on its own → fast scrollable
	•	PAGE BODY CONTENT → full page scroll → non-fast scrollable, needs main thread to check things like overflow, sticky, layout, JS listeners → compositor cannot scroll it independently

⸻

Real-world analogy
	•	Fast scrollable → elevator that moves automatically, you just push the button
	•	Non-fast scrollable → elevator that needs operator approval each time → slower if operator busy (main thread)

⸻

Why this matters
	•	If you scroll a non-fast region:
	•	Compositor has to ask main thread: “Can I scroll? Will JS prevent this?”
	•	If main thread is busy → scroll lags → jank
	•	Chrome tries to coalesce / prioritize to reduce delay
	•	If you scroll a fast region:
	•	Compositor moves it immediately
	•	No main-thread delay → buttery smooth scrolling

⸻

Code to experiment with

<div style="height:200px; overflow: scroll; border:1px solid red;">
  <!-- Fast scrollable div -->
  <p>Lots of content here...</p>
</div>

<div>
  <!-- Non-fast scrollable region -->
  <p>Full page content...</p>
</div>

<script>
  document.addEventListener("scroll", e => {
    console.log("Scroll event fired!", e.target);
  }, { passive: true });
</script>

	•	Try scrolling the red-bordered div → compositor scrolls fast
	•	Scroll the page body → main thread involvement → slower if JS busy

⸻

If you want, I can draw a diagram showing the compositor thread asking the main thread for non-fast scrollable regions vs fast scrollable regions, so you can see exactly what’s happening under the hood.

Do you want me to make that diagram?