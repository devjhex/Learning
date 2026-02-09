	What confused me at first?
	•	What clicked?
	•	What mental model changed?
	•	What surprised me?

	What clicked?
	### Role of the renderer process
	1. The role of the renderer process is to turn your HTML, CSS and JS into a web page that users that interact with.

	### Parsing
	1. When commit navigation starts and then renderer is ready parsing begins.
	2. Parsing of the HTML into the DOM, the CSS to CSSOM and the Javascript to its own abstract layer tree.

	### Subresource loading
	1. The page does have external resources like css, Js, images, etc so it could load those resources while parsing to build the DOM. But the preload scanner peaks at tokens genereated by the HTML parser and sends requests to the network thread in the browser process.

	### Javascript can block the main thread.
	1. When the parser finds a script tag it has to pause parsing so as to load, parse and execute the JS and this is because it could change the shape of the document which could change the DOM as well.

    ### Hint to the browser what you want to load.
	1. We could send hints to the browser telling it how to load our external resources. We could use async, defer, rel = "preload". More on that you can check in resources prioritization.

	### Style calculation
	1. Having the DOM is not enough so as to paint the page on the screen, so the main thread has to go through the DOM and create computed styles of each and every element depending on what was written in your CSS. And even if there is no CSS there are default styles for every element in the Page.

	### Layout
	1. Still having the DOM and the computed styles is not enough to paint the elements on the page, because we still have no idea where the elements are drawn how are they drawn so the main thread goes through the DOM and computed styles to create something called the Layout tree

	NOTE: 
	Determining the Layout of a page is a challenging task. Even the simplest page layout like a block flow from top to bottom has to consider how big the font is and where to line break them because those affect the size and shape of a paragraph; which then affects where the following paragraph needs to be.
	CSS can make element float to one side, mask overflow item, and change writing directions. You can imagine, this layout stage has a mighty task. In Chrome, a whole team of engineers works on the layout. If you want to see details of their work, few talks from BlinkOn Conference are recorded and quite interesting to watch.

	### Paint
	1. Still having the DOM, the computed styles and the layout tree is still not enough because you have to know in what order you have to draw these elements. So the main thread goes through the DOM, the computed styles and the layout tree so as to create something called paint records so as to know how to paint the elements in terms of order for example z-index of some elements etc.

	2. You have to understand that in all these stages, the DOM, the computed styles, the layout, the paint, the result of the previous operation is used to create new data. For example if something changes in the layout tree, then the paint order needs to be regenerated for the affected parts of the document.

	3. If you are animating elements, the browser has to run these operations in between every frame. Most of our displays refresh the screen 60 times a second (60 fps); animation will appear smooth to human eyes when you are moving things across the screen at every frame. However, if the animation misses the frames in between, then the page will appear "janky".

	4. You can also divide JS operations into small chunks and schedule them to run at every frame using something like requestAnimationFrame()

	### Compositing
	1. Compositing is a technique  to separate parts of a page into layers, rasterizes them separately and composites them as a page in the compositor thread. If a scroll happens, since the layers are already rasterized all it has to do is to composite a new frame. Animation can be achieved in the same way by moving layers, and compositing a new frame.

	### Dividing into layers
	1. So when it comes to creating layers the main thread goes through the layout tree to create the layer tree and this is to find out which elements need to become layers. 
	2. If certain parts of the page that should be separate layers are not gettinga layer then you can let the browser know of that by hinting to it with the will-change attribute in CSS
	3. Dont be tempted to give layers to every element in the page just because you want to make sure you are using the compositor thread because it may be faster to rasterize smaller parts of the page than rasterizing everything leave some work for the main thread.

	### Raster and composite off of the main thread
	1. Once the layer tree is created and the paint records are got, the main thread commits that information to the compositor thread.
	2. The compositor thread then rasterizes each layer. A layer could be large like the entire length of the page so the compositor thread then divides them into tiles and sends each tile off to the raster threads. Raster threads rasterize each tile and stores them in GPU memory.
	3. The compositor thread can prioritize which raster threads that should rasterize first or earlier like stuff that is within the viewport could be rastered first.
	4. A layer could have multiple tilings for different resolutions like for zoom in action.
	5. Once tiles are rastered the compositor gathers tile information called draw quads  to form a compositor frame.

	Draw quads : contain info like tile's locaiton, where it is to be drawn on the page putting into consideration the page compositing
	Compositor frame: Is a collection of draw quads that represent a frame of a page.

	6. A compositor frame is then submitted to the browser process via IPC.
	7. At this point, another compositor frame could be added from UI thread for the browser UI change or from other renderer processes for extensions. 
	8. These compositor frames are sent to the GPU to display it on a screen. If a scroll event comes in, compositor thread creates another compositor frame to be sent to the GPU.hese compositor frames are sent to the GPU to display it on a screen. If a scroll event comes in, compositor thread creates another compositor frame to be sent to the GPU.

	What mental model changed?
	I now understand how the compositor thread really works and its not just a fairy tale that i could just send work to it and it helps the main thread, its really something very important.

	What surprised me?
	The model how the compositor thread and the raster threads actually communicate with the GPU it's just amazing.




