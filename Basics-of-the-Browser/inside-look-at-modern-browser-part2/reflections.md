	What confused me at first?
	•	What clicked?
	•	What mental model changed?
	•	What surprised me?

    What clicked?
    1. The UI thread handles the input, it also sends the input to the network thread.

    2. The network thread makes sure the request is returned and handles all the network stuff, it also checks to see whether what is returned is safe like, making sure it is the right type of data (MIME sniffing) etc This is also where Safe browsing is done and also Cross origin Read blocking.

    3. The network thread might figure that it might be redirected and it may tell the UI thread that it needs to make another network request. This is done when a 301 http response is brought back.

    4. Once the network thread is confident that all is good then it tells the UI thread and then the UI thread looks for a renderer process to take the response and load it.

    5. Since this might take time there is an optimization process that speeds up the looking for renderer process while getting the response from the network thread it is called "Renderer process pre warming"

    6. Then the commit navigation happens. This is where the browser process confirms the renderer process to load new content.

    7. After the commit navigation you see the address bar is updated and the security indicator and site settings UI reflects the site information of the new page.

    8. When the renderer finishes loading, it tells the browser process via IPC that it has finished loading when the onload event has been fired on all frames of the page and have finished executing. That's when the spinner stops spinning (pretty cool)

    9. If the user navigates to different site, the same steps will happen but in the opposite direction like from the renderer process to the browser process. It will also check with the current renderer process if they care about the beforeunload event.

    10. When a navigation happens, network thread checks the domain against registered service worker scopes, if a service worker is registered for that URL, the UI thread finds a renderer process in order to execute the service worker code.

    11. Navigation Preload is a mechanism to speed up this process by loading resources in parallel to service worker startup.

    2. What mental model changed?
    What changed for me is that there are more threads than i thought like the UI thread and the network thread and the other threads just aside the Main thread, compositor thread, the worker thread and the raster threads.

    3. What suprised me was about the step after the commit navigation where the renderer process communicates via IPC to the browser process that is has finished loading then the spinner stops moving i thought that was pretty cool.