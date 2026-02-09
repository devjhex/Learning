What is navigation commit?
A navigation commit is when the browser has actually found a renderer process to take on the html from the network thread so as to loaded on the screen for the user.

Answser : At this point, address bar is updated and the security indicator and site settings UI reflects the site information of the new page. The session history for the tab will be updated so back/forward buttons will step through the site that was just navigated to. To facilitate tab/session restore when you close a tab or window, the session history is stored on disk.

When does rendering actually happen?
Rendering happens when browser finds a renderer process finds a renderer process for the data that has been returned by the network thread.

Why doesn’t JS stop after onload?
JavaScript doesnt stop onload because its execution could still lead up to rendering of more pages or more parts on the screen etc

Why does the browser stop the spinner?
The browser stops the spinner on the screen when the javascript has finished loading  and all the onload events have been fired on all pages of the website that has been requested by the user.

Answer : When the renderer finishes rendering, it means the initial page load has completed (all frames fired onload). The renderer notifies the browser process via IPC so the UI can stop the loading spinner. Rendering may continue afterward due to client-side JavaScript.

What is service worker?
A service worker is a separate thread that could load work like javascript off of the main thread to do things like caching or even fetching some data desired by the user.

Answer : Service worker is a way to write network proxy in your application code; allowing web developers to have more control over what to cache locally and when to get new data from the network.

What is navigation preload?
Navigation preload is an optimization mechanism that enablbes the page being loaded to be loaded in parallel to the service worker since the service worker would need to wake up first then the loading of the page could actually happen, now it wakes up while the page is loading.

Anwser : Navigation Preload is a mechanism to speed up this process by loading resources in parallel to service worker startup.

What is the beforeunload event?
A beforeunload event is an event fired when a navigation to other page is taking place like leaving a page from a another page as a result of clicking a link to another page etc.