# Web Workers & Service Workers - Summary

## Web Workers
- Run JavaScript in a background thread, separate from the main UI thread
- Used for heavy computations or tasks that would block the UI (e.g., data processing, parsing, image manipulation)
- No direct access to DOM; communicate with main thread via messages
- Improves performance and keeps UI responsive

## Service Workers
- Special type of worker that acts as a proxy between the web page and the network
- Runs in the background, even when the page is closed
- Used for offline support, caching, background sync, push notifications
- Can intercept and handle network requests, enabling PWAs and advanced caching strategies

## Key Differences
- **Web Workers:** For offloading computation; keep UI smooth
- **Service Workers:** For controlling network, enabling offline/advanced web app features
- Both run off the main thread, but Service Workers have lifecycle and network control
