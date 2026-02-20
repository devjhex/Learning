# Key topics

- [ ]  What is a reflow(layout)?
    - A reflow is when the browser recalculates the layout and position of elements on the page.

- [ ]  When does it happen?
    - It happens properties or styles of elements that affect the layout have been changed. like width, height, font-size, margin padding etc


- [ ]  What part of the rendering process is involved?
    - the layout phase of the rendering process.

- [ ]  What changes trigger it?
     - changes of width, height, font-size, margin, padding etc.


### repaints

- [ ]  What is a repaint?
    - A repaint is when the browser redraws elements on the page

- [ ]  When does a repaint happen?
    - A repaint happens when elements that dont affect the layout of the page have been changed like background color, outline, visibility etc

- [ ]  What part of the rendering pipeline is involved?
    - A repaint happens in the paint phase.

- [ ]  What kinds of changes trigger a repaint
    - the change of color, background-color, outline, visibility etc


 **Layout thrash / forced synchronous layout**

1. What is layout thrashing?
    - layout thrashing is when the there are a series of layout/reflow events that are happening repeatedly and synchronously  resulting in slow fps

2. What causes layout thrashing?
    - When you repeatedly read and write to the DOM in loops (e.g., reading `offsetWidth` then changing styles repeatedly)


3. Which DOM reads trigger layout thrashing?
    - Reads of offsetHeight
    - element.offsetWidth
    - element.getBoundingClientRect()
    - element.clientWidth
    - window.getComputedStyle(element)

4. Which DOM writes trigger layout thrashing?
    ```
    element.style.width = "200px"     // write (dirty layout)
    element.offsetWidth  // read (forces layout NOW)
    element.style.height = "300px" // write (dirty again)
    element.offsetHeight // read (forces layout AGAIN)

    ```
5. How does layout thrashing affect performance?
    - Main-thread-blocking operation; causes janky/laggy user experience; may increase CLS and FCP times

6. How can layout thrashing be prevented?
- **Measure once:** Read DOM properties once and store the value before modifying
- **Batch operations:** Group all DOM reads together, then all DOM writes
- **Edit deep in the DOM tree:** Make changes to the lowest-level element possible to limit cascade effects

- **Use `requestAnimationFrame`:** Schedule visual changes at the start of a new frame

- **Use CSS transforms:** For animations, prefer `transform` and `opacity` over properties like `width`, `height`, `top`, `left`

- **Remove from DOM:** For multiple edits, remove element, batch edit, then re-add

- **Promote to layers:** Use `will-change`, `transform: translateZ(0)`, `position: absolute` for frequently changing elements