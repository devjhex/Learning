# Key concepts from inside look at a modern browser part 2

## It all starts from the BROWSER PROCESS
### everything outside of the tab is handled by the browser process
- Everything outside the tab (renderer process) is handled by the browser process.
### the browser process has other threads that help the proper functioning of the browser.
- The browser process has threads like the UI thread  which draws buttons and input fields of the browser and the network thread which deals with the network stack and recieving data from the internet, there is also the storage thread which controls access to files and more.
### input from the user is handled and initiated by the browser process
- When you type a URL into the address bar, your input is handled by the UI thread of the browser process.

## An image showing how the browser process has and usese more threads.
![The browser process with some of its threads](images/browser-processes-1abcf214c73ef_1920.png)

## Simple navigation
### HANDLING INPUT(1)
- When a user types something in the address bar the UI thread asks "Is this a search query or a URL?"
- In chrome the address bar is also a search bar.
- The UI thread needs to parse and find out whether to send you to a search engine or to the site you requested.

![The UI thread asking hmmmm is this a search query or a URL](images/UI-thread-asking.png)

### START NAVIGATION(2)
- When the user hits the enter key the UI thread initiates a network call to get the site content.

## network thread
- The network thread is the one who will handle that and it will go through appropriate protocols like the DNS look up and establishing TLS connection for the request.

![The UI thread asking the Network thread to make a request since it handles network stuff](images/UI-thread-asking.png)

## NOTE
- At this point the network thread may recieve a server redirect like HTTP 301 and in this case the network thread may communicate with the UI thread that the server is requesting a redirect. Then another URL request will be initiated.

### READ RESPONSE (3)
- Once the response body (payload) starts to come in, the network thread looks at the first few bytes of the stream if necessary.
## Content-type header sometimes is missing
- The response's Content-Type header should say what type of data it is, but since it may be missing or wrong, MIME Type sniffing is done here.

![the nature of the payload](images/payload.png)

#### MIME type sniffing
- In the absence of a MIME type, or in certain cases where browsers believe they are incorrect, browsers may perform MIME sniffing — (guessing the correct MIME type by looking at the bytes of the resource.)

## But if payload is HTML
- Then the data will be passed on to the renderer process.
## But if payload is a zip file.
- Then the data will be passed on to the download manager.

![Network thread checking if the payload is safe](images/is-payload-safe.png)

## NOTE
- This is also where SafeBrowsing check happens hence if its a malicious site then a warning page is displayed.
- Also Cross-Orign-Read-Blocking (CORB) check happens in order to make sure cross site data does not make it to the renderer process.
## CORB
- Cross-Origin Read Blocking (CORB) is a new web platform security feature that helps mitigate the threat of side-channel attacks (including Spectre). It is designed to prevent the browser from delivering certain cross-origin network responses to a web page, when they might contain sensitive information and are not needed for existing web features. For example, it will block a cross-origin text/html response requested from a <script> or <img> tag, replacing it with an empty response instead. This is an important part of the protections included with Site Isolation.

### FIND A RENDERER PROCESS (4)
- Once all checks are done and the network thread is confident that the browser should navigate to that site, it tells the UI thread that the data is ready.
- The UI thread then looks for a renderer process to carry on the rendering of the webpage.

![find a renderer process](images/find-renderer-process.png)

## NOTE
- Since the network request could take several hundred milliseconds to get a response back, an optimization to speed up this process is applied. When the UI thread is sending a URL request to the network thread at step 2, it already knows which site they are navigating to. The UI thread tries to proactively find or start a renderer process in parallel to the network request. This way, if all goes as expected, a renderer process is already in standby position when the network thread received data. This standby process might not get used if the navigation redirects cross-site, in which case a different process might be needed.
- The process is normally called "Renderer Process Pre-warming" or "Early speculative renderer creation"

### COMMIT NAVIGATION
- Now that the data and the renderer process is ready, an IPC is sent from the browser process to the renderer process to commit the navigation. It also passes on the data stream so the renderer process can keep receiving HTML data. Once the browser process hears confirmation that the commit has happened in the renderer process, the navigation is complete and the document loading phase begins.
- At this point, address bar is updated and the security indicator and site settings UI reflects the site information of the new page. The session history for the tab will be updated so back/forward buttons will step through the site that was just navigated to. To facilitate tab/session restore when you close a tab or window, the session history is stored on disk.

![find a renderer process](images/commit-navigation.png)

### EXTRA INITIAL LOAD STEP

- When the renderer finishes rendering, it means the initial page load has completed (all frames fired onload). The renderer notifies the browser process via IPC so the UI can stop the loading spinner. Rendering may continue afterward due to client-side JavaScript.

![An IPC from the renderer proess to the browser process to notify the page has loaded.](images/page-finish-loading-1bee6888e9e56_1920.png)

### NAVIGATING TO A DIFFERENT SITE
- Now what happens if the user tries to navigate to a different site. Well the same steps that happened before from the browser process to the renderer process but before that it will first check with the currently rendered site to see if they care about the beforeunload event.

## NOTE
Caution: Do not add unconditional beforeunload handlers. It creates more latency because the handler needs to be executed before the navigation can even be started. This event handler should be added only when needed, for example if users need to be warned that they might lose data they've entered on the page.

![The browser process makes sure the currently rendered site cares about the beforeunload event](images/beforeUnload.png)

- if the navigation was initiated from the renderer process that is the user clicked a link on the site or client side js has run something like (window.location = "https://jhex.com)

## NOTE
When the new navigation is made to a different site than currently rendered one, a separate render process is called in to handle the new navigation while current render process is kept around to handle events like unload.

### INCASE OF A SERVICE WORKER
- Service worker is a way to write network proxy in your application code; allowing web developers to have more control over what to cache locally and when to get new data from the network. If service worker is set to load the page from cache then there is no need to request the data from the network.

The important part to remember is that service worker is JavaScript code that runs in a renderer process. But when the navigation request comes in, how does a browser process know the site has a service worker?


![is service worker installed](images/isServiceWorkerInstalled.png)

- When a service worker is registered, the scope of the service worker is kept as a reference (you can read more about scope in this The Service Worker Lifecycle article). When a navigation happens, network thread checks the domain against registered service worker scopes, if a service worker is registered for that URL, the UI thread finds a renderer process in order to execute the service worker code. The service worker may load data from cache, eliminating the need to request data from the network, or it may request new resources from the network.

### NAVIGATION PRELOAD
- ou can see this round trip between the browser process and renderer process could result in delays if service worker eventually decides to request data from the network. Navigation Preload is a mechanism to speed up this process by loading resources in parallel to service worker startup. It marks these requests with a header, allowing servers to decide to send different content for these requests; for example, just updated data instead of a full document.

![is service worker installed](images/navPre.png)



