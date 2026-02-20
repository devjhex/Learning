# Key concepts. 

## Reflow
- **Definition:** The browser recalculates the layout and position of elements on the page

- **Reflow happens:** When properties that affect layout change (e.g., `width`, `height`, `font-size`, `margin`, `padding`)


## What is Repaint? OR REDRAW
- **Definition:** The browser redraws elements on the page.

- **Repaint happens:** When properties that don't affect layout change (e.g., `color`, `background-color`, `outline`, `visibility`)


## What is Layout Thrashing?
- **Definition:** A series of layout/reflow events happening repeatedly and synchronously, resulting in slow frame rendering times (low FPS)

- **When it happens:** When you repeatedly read and write to the DOM in loops (e.g., reading `offsetWidth` then changing styles repeatedly)

## What is Layout Shift?
- **Definition:** A noticeable shift of elements within the viewport when layout changes

- **Unwelcome layout shift:** When a layout shift results in a bad visual experience (user sees elements jumping around)



### Layout Avalanche Effect
- **What it is:** When changing one element's layout causes a cascade of layout changes in other elements

- **Example:** Hovering over a grid item to resize it pushes all other items, causing them to reflow