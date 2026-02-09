# Input coming to the compositor

## The browser receives a user gesture
- When user gesture like touch on a screen occurs, the browser process is the one that receives the gesture at first.
- However, the browser process is only aware of where (co-ordinates) that gesture occurred since content inside of a tab is handled by the renderer process.
- So the browser process sends the event type (like touchstart) and its coordinates to the renderer process.

## The renderer process receives the event from the browser process
- Renderer process handles the event appropriately by finding the event target and running event listeners that are attached.

![The browser process sends the input event to the renderer process](images/input-event.png)

## Compositor receieves input events
- Compositor thread can create a new composite frame completely independent of the main thread. But what if some event listeners were attached to the page? How would the compositor thread find out if the event needs to be handled?

## Non-fast scrollable region
- Since running JavaScript is the main thread's job, when a page is composited, the compositor thread marks a region of the page that has event handlers attached as "Non-Fast Scrollable Region".
- By having this information, the compositor thread can make sure to send input event to the main thread if the event occurs in that region. 
-  If input event comes from outside of this region, then the compositor thread carries on compositing new frame without waiting for the main thread.

IMAGE

## Be aware of how you write event handlers
- A common event handling pattern in web development is event delegation. Since events bubble, you can attach one event handler at the topmost element and delegate tasks based on event target. You might have seen or written code like the below.

```

document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
        event.preventDefault();
    }
});

```

- Attractive right???
- However, if you look at this code from the browser's point of view, now the entire page is marked as a non-fast scrollable region. 
- This means even if your application doesn't care about input from certain parts of the page, the compositor thread has to communicate with the main thread and wait for it every time an input event comes in. Thus, the smooth scrolling ability of the compositor is defeated.

IMAGE

- In order to mitigate this from happening, you can pass passive: true options in your event listener. This hints to the browser that you still want to listen to the event in the main thread, but compositor can go ahead and composite new frame as well.


```

document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
        event.preventDefault();
    }
}, {passive:true});

```
## Check if the event is cancellable

IMAGE

- Imagine you have a box in a page that you want to limit scroll direction to horizontal scroll only.
- Using passive: true option in your pointer event means that the page scroll can be smooth, but vertical scroll might have started by the time you want to preventDefault in order to limit scroll direction. 
- You can check against this by using event.cancelable method.

```
document.body.addEventListener('pointermove', event => {
    if (event.cancelable) {
        event.preventDefault(); // block the native scroll
        /*
        *  do what you want the application to do here
        */
    }
}, {passive: true});

```

- Alternatively, you may use CSS rule like touch-action to completely eliminate the event handler.

```
#area {
  touch-action: pan-x;
}
```

### Finding the event target

IMAGE

- When the compositor thread sends an input event to the main thread, the first thing to run is a hit test to find the event target.
- Hit test uses paint records data that was generated in the rendering process to find out what is underneath the point coordinates in which the event occurred.

### Minimizing evnt dispatches on the main thread
- In the previous post, we discussed how our typical display refreshes screen 60 times a second and how we need to keep up with the cadence for smooth animation. For input, a typical touch-screen device delivers touch event 60-120 times a second, and a typical mouse delivers events 100 times a second. Input event has higher fidelity than our screen can refresh.

- If a continuous event like touchmove was sent to the main thread 120 times a second, then it might trigger excessive amount of hit tests and JavaScript execution compared to how slow the screen can refresh.

IMAGE

Events flooding could cause page jank

- To minimize excessive calls to the main thread, Chrome coalesces continuous events (such as wheel, mousewheel, mousemove, pointermove, touchmove ) and delays dispatching until right before the next requestAnimationFrame.

- Any discrete events like keydown, keyup, mouseup, mousedown, touchstart, and touchend are dispatched immediately.

### Use getCoalescedEvents to get intra-frame events
- For most web applications, coalesced events should be enough to provide a good user experience. However, if you are building things like drawing application and putting a path based on touchmove coordinates, you may lose in-between coordinates to draw a smooth line. In that case, you can use the getCoalescedEvents method in the pointer event to get information about those coalesced events.

IMAGE

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
