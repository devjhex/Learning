# Reflows, Repaints and Layout Thrashing / Forced synchronous layout

## Quick Summary

### What is Reflow?
- **Definition:** The browser recalculates the layout and position of elements on the page

- **When it happens:** When properties that affect layout change (e.g., `width`, `height`, `font-size`, `margin`, `padding`)
- **Impact:** High performance cost; can trigger entire page reflow

### What is Repaint? OR REDRAW
- **Definition:** The browser redraws elements on the page
- **When it happens:** When properties that don't affect layout change (e.g., `color`, `background-color`, `outline`, `visibility`)
- **Impact:** Lower cost than reflow, but still affects performance

### Key Difference
- **Reflow:** Affects layout → larger performance impact
- **Repaint:** Affects visual appearance only → smaller performance impact


### What is Layout Thrashing?
- **Definition:** A series of layout/reflow events happening repeatedly and synchronously, resulting in slow frame rendering times (low FPS)
- **Also called:** Forced reflow or uninterruptible reflow
- **When it happens:** When you repeatedly read and write to the DOM in loops (e.g., reading `offsetWidth` then changing styles repeatedly)
- **Impact:** Main-thread-blocking operation; causes janky/laggy user experience; may increase CLS and FCP times
- **Example:** Reading an element's height and changing margin in a loop triggers reflow on every iteration

### What is Layout Shift?
- **Definition:** A noticeable shift of elements within the viewport when layout changes
- **Unwelcome layout shift:** When a layout shift results in a bad visual experience (user sees elements jumping around)
- **Impact:** Confusing for users; affects perceived page quality
- **Related metric:** CLS (Cumulative Layout Shift) - measures visual stability

### Layout Avalanche Effect
- **What it is:** When changing one element's layout causes a cascade of layout changes in other elements
- **Example:** Hovering over a grid item to resize it pushes all other items, causing them to reflow
- **Solution:** Use `transform: scale()` instead of changing width/height; use `position: absolute` for frequently changing elements




# HOW TO AVOID
### Best Practices to Minimize Layout Thrashing
- **Measure once:** Read DOM properties once and store the value before modifying
- **Batch operations:** Group all DOM reads together, then all DOM writes
- **Edit deep in the DOM tree:** Make changes to the lowest-level element possible to limit cascade effects
- **Use `requestAnimationFrame`:** Schedule visual changes at the start of a new frame
- **Use CSS transforms:** For animations, prefer `transform` and `opacity` over properties like `width`, `height`, `top`, `left`
- **Remove from DOM:** For multiple edits, remove element, batch edit, then re-add
- **Promote to layers:** Use `will-change`, `transform: translateZ(0)`, `position: absolute` for frequently changing elements



### How to Avoid reflows and repaints
- Use `transform` instead of changing position
- Use `opacity` instead of visibility changes
- Use `will-change` to isolate elements to separate layers
- Minimize frequent DOM operations
