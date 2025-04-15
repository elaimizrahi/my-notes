### Event Types 
Device Input Events 
-  Keyboard (e.g. key press, key release)
- Pointing (e.g. move move, button press, button release) 
Window Input Events 
- Changes (e.g. resize, closing) 
Window or Widget Events 
- Pointing (e.g. mouse enter, mouse leave) 
- Focus (e.g. focus gained, focus lost)
System Events 
- Timer (e.g. tick, completed) 
Application Events 
- Thread (e.g. progress, completed, …)

We keep track of these events with an event queue (FIFO)

Each event is created when the state changes, for example a `keydown` changes from up to down


#### Window Dispatch
- The system maintains a list of windows ordered from back to front, usually sends event to focused window (unless it's a global hook)
- The Windowing System will constantly be polling the I/O devices, waiting for one to update accordingly where it will send an event to the system
- The `UI Toolkit` will translate the event into its proper action

### Simplekit events
- `SKEvent` is the global type for events, this is the first type received when processing an event
- In simplekit we also have `SKMouseEvent`, `SKKeyboardEvent`, and more
- Can translate this properly below:
  ```jsx
export const fundamentalTranslator = { 
	update(fe: FundamentalEvent): SKEvent { switch (fe.type) { 
		case "mousedown": 
		case "mouseup": 
		case "mousemove": 
			return new SKMouseEvent(fe.type, fe.timeStamp, fe.x || 0, fe.y || 0); 
			break;
		...
	...
}
addSKEventTranslator(fundamentalTranslator);
startSimpleKit();
```

![[Pasted image 20250225123239.png]]

### Hit Testing
Hit testing checks if an event happens inside (or on the edge) of a shape
example hit test function:
```jsx
function insideHitTestRectangle( mx: number, my: number, x: number, y: number, w: number, h: number ) { 
	return mx >= x && mx <= x + w && my >= y && my <= y + h 
}
```
For edge hit testing, we just add another `const s = strokeWidth/2` for example as the `edge width` 
- We check that the `mx` and `my` are outside the shape after adding the stroke, but that they are **not** inside the shape after subtracting the stroke
- For example for circle edge hit test, the edge hit is true when distance is in range [r – s/2, r + s/2]
- We can also do a **polyline** hit test which is just checking this hit test for every line on the shape's edge
	- Calculate closest point on line segment `(qx, qy)` to event point `(mx, my)`
	- Edge hit is true when $dist < s/2$
	- Use scalar projection of the line $m$ onto $q$
	- For inside polygon hit test:
	  ![[Pasted image 20250225125846.png]]
	- Special Case:	![[Pasted image 20250225125906.png]]
- This can be computationally intesive, so we can:
	- Use offscreen buffer to draw shape
	- Transform mouse coordinates to match buffer
	- and more
### Other Hit-Test Selection Paradigms
Text selection 
- insertion point, drag to select
Crossing intersection 
- select by drawing stroke through shapes 
Shape Intersection
- Marquee selection (select shapes in oriented bounding box) 
- Lasso selection (select shapes enclosed in freeform path)