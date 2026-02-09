

### The Browser process gets the user event but only knows its co-ordinates nothing else
- When a user does a scroll or a drag, or a touch whatever high frequency event, the browser recieves it but it only knows where it happened

### The browser process sends it the renderer process
- The browser process sends the event to the renderer process.
- The renderer process then sends it to the compositor thread, the compositor thread is going to make a journey.

### If the event is one of JS's best friends (no-fast scroallable region)
- The compositor thread then checks whether its one of JS's best friends and if it is, it asks JS "Hey do you need something from this friend of yours (event)? "
- If JS does need it it will have to get it from the event, If it doesnt then JS will say no and the journey will continue.
- We can tell the JS that there is nothing he will get from some of his friends to prevent wasting time. {passive : true}
- We dont want to make the area full of his friends because that means when ever the compositor finds one of his friends then he will always ask JS about him or her and this will waste time.
- Some of his friends are non negotiable so they HAVE to do what they are supposed to do no skipping, no excuses.


## Some of the things that JS will do with these friends of his
    - Hit testing
    - Event coalescing
    
