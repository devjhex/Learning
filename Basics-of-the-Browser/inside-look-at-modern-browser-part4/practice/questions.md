Event flow and threads
1. What happens when an input event occurs (pointer, touch, wheel, scroll)?
    - When an input event occurs like a pointer, touch, whell or scrolll, it is first sent to the browser process which only has the co-ordinates of where the event happened from.
    - Then the browser process sends the event with it's co-ordinates to the appropriate renderer process which then runs them and also the code that is to be run after these events are fired.

2. Which threads are involved: browser process, compositor thread, main thread?
    - Compositor thread, and main thread.

3. How does the compositor handle input without blocking the main thread?
    - The code that is run after the input is fired should be passive and when it is passive it will let the main thread know that whatever is in the event listener will not block the main thread so it could continue with its job that is maybe compositing frames.


Cancelable vs passive
1. What does event.cancelable mean?
    - event.cancelable is a property on events that returns true of false as to whether the event is cancelable or not and that is whether its default behavior could be stopped completely or not.
    
2. What happens if you call preventDefault on a non-cancelable event?
    - An error could be returned in the console that you cannot change or cancel the non-cancelable event and your code would be ignored.

3. When should you use {passive: true}?
    - You should use {passive: true} when you need to help the compositor thread assure the main thread that the code in the event listener still wants to be listened to but we could also go ahead and continue to composite new frames as well.

4. Why do passive listeners improve scrolling performance?
    - Because they allow the compositor thread to go ahead and continue compositing new frames immediately even when the listeners still have to be listened to, the compositor thread assures the main thread that the code will not call prevent Default (block the scrolling).

5. How does CSS touch-action relate to passive events?
    - For this CSS touch-action says to the browser "or this element, scrolling behavior is already decided." and there is no reason to even ask JS at all.


Minimizing main thread dispatches
1. Why is it bad to dispatch high-frequency events directly on the main thread?
    - It is dangerous because since they are high frequency they will be dispatched so many times in each frame  and there would be an excessive amount of hit tests and JS execution compared to how slow the screen can refresh.

2. Which events are considered high-frequency (mousemove, wheel, touchmove, etc)?
    - mousemove, wheel, touchmove, pointermove, mousewheel etc.

3. What is event coalescing and why is it done?
    - Event Coalescing is waiting to dispatch a high frequency event until right before the next requestAnimationFrame before delivering them to JS.
    Why ...
    - JS is slower than input devices.
    - To protect the main thread
    - To align events with rendering

4. How does the browser decide when to dispatch coalesced events?
    - The browser normally dispatches coalesced events right before the next frame (requestAnimationFrame)


requestAnimationFrame(rAF)
1. What is rAF and why does it matter for input events?
    - requestAnimationFrame is the browser saying:...

    “Hey JS, if you want to change anything visual, do it now, we’re about to draw.”
    ```
    1. Input processing
    2. JS (event handlers, promises, etc.)
    3. requestAnimationFrame callbacks   👈 THIS
    4. Style calculation
    5. Layout
    6. Paint
    7. Compositing
    8. GPU presents frame

    ```
    Why doest it matter for input events?
    - Input events are  collected and coalesced until the browser reaches rAF.


2. How does Chrome dispatch coalesced events just before the next rAF?
    STEPS:
    A. Event arrives (your mouse, touch or scroll, drag, etc)

    B. Coalescing happens 
    1.(Chrome sees a high frequency event)

    2.Instead of sending it JS immediately...
    (It buffers it)
    (It keeps the latest coordiates/state)
    (it drops redundant intermediate events for efficiency)

    C. Waits for the right moments
    (Coalesced events are held until the browser is about to calculate the next frame.)

    D. Dispatch to JS
    (Chrome delivers the coalesced event just in time)
    (Only the latest event data)
    (Only once a frame)
    This ensures your visual updates happen once per frame, not dozens of times.

3. How does using rAF ensure smooth animations and scrolling?
    - rAF ensures smooth scrolling by synchronizing JS updates with the browser’s frame rendering, so input changes are applied once per frame, preventing main-thread overload and dropped frames.

Event handling best practices
1. When should you use cancelable vs passive listeners?
    -I should use cancelable vs passive listeners when i would want to give immediate scrolling for the compositor and for defensive programming for whether an event is not cancelable instead of getting errors from the browser.

2. How can you avoid scroll jank in your JS?
    - By dividing your JS with the rAF that is having smaller chunks of JS.

3. What's the relationship between compositor scrolling and main thread JS execution?
    - The relationship is the compositor is used to composite new frames but some of the layers it actually composites have event listeners that have to be fired by JS and that is done by the Main thread so the 2 have to communicate as to when there's no need for JS to intervene or not.

4. How does event delegation affect high frequency event handling?
    - Event delegation creates a large non-fast scrollable region which makes the compositor have to ask the main thread all the time since the event listener is at a high level. This defeats the role of the compositor which came to make things faster.


Deeper understanding
1. What is a non-fast scrollable region and how does it affect event dispatch?
    - Non-fast scrollable region is the region that has to make sure to send the main thread once and event happens in that region.

2. How does the browser decide which input events can bypass the main thread?
    - The browser sees whether its is a non-fast scrollable region, It also sees whether the event passive, also whether the event is cancelable.

3. Why checking event.cancelable can be considered defensive programming?
    - Not all events are cancelled.
    - Avoid runtime or wasted effort
    - Browser performance benefit

    Checking event.cancelable is defensive programming because it ensures preventDefault() is only called when valid, preventing errors and avoiding unintended performance impacts.

Can you answer these questions?
1. Why do Chrome tabs crash individually?
    - They crash individually because each tab has its own renderer process

2. Why isolate origins into separate renderers?
    - To enforce Same-Orign Policy

3. When does a browser send multiple requests for a single URL?
    -  when the original request cannot be fulfilled directly and requires additional fetches. Like a redirects, service workers etc
