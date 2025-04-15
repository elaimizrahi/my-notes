We want a responsive feeling UI that seems like it responds in a timely manner

We can do this by:
- designing for human perception + expectations
- using async exectution

**10ms** is the minimal time to be affected by visual stimulus
**140ms** is the maximal time there should be between cause-effect events
**1s** is the visual-motor reaction time
**10s** is the time to prepare for conscious cognition task

We can use busy indicators (spinning wheel for example) for operations that take a few seconds longer than expected

If the expected time is higher, progress bars are a strong visual cue to the user

**Progress Loading** allows user to adjust to layout and makes loading seem faster
- starts by rendering the grid, then each element as they come in
- Could also be how it shows blurry images before eventually showing full resolutions

**Predicting next operation** - use periods of low load to pre-compute responses to high probability requests (load in background)

**Asynchronous execution** - Execute tasks independently from the main program flow
- do more than one thing at once
- with async programming or threading

**Async programming** 
- Allows for execution of tasks without blocking in a **single thread** 
	- NOT concurrent execution
	- promise/future, coroutines, nonblocking I/O
	- Input events are async methods which are handled as callbacks bound to a DOM element
	- Fetch API starts fetching a resource, returns a promise object with a `pending`, `resolved`, and `rejected` state based on the current response

**Handling Non-Web-API Long Tasks**
We want to keep the UI responsive, provide feedback, and allow it to be cancelled

**Threading**
Manage multiple concurrent threads with shared resources but different instructions
- divides computation, reduces locking
- risks though with race conditions
- browsers support this with 
	- dedicated workers
	- shared workers
	- service workers
