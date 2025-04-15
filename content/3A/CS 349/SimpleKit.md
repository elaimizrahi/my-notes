We use an MVC model to draw objects with simplekit

This allows the DOM to only update the view when the model updates

![[Pasted image 20250225121229.png]]

### Windows
Windows provide a graphical layer between the MVC and the user
Provides 3 services:
1. Manage list of windows (resize, focus, create, etc)
2. Provide each application with independent drawing area
3. Dispatch low-level inputs to the focused window
Uses drawing canvas abstraction
- `bitblt` (bit block transfer) is method of rendering at right position

### UI Toolkit (SimpleKit)
A UI toolkit provides a level of abstraction for programmers so that they can draw shapes + utils like buttons easily
- `SetPixel`, `Stroke`, `Region`, `DrawLine`
We maintain a graphical context, which can then be used to easily update
![[Pasted image 20250225121727.png]]

#### Creating a simplekit canvas + drawing
```jsx
import { startSimpleKit, setSKDrawCallback } from "simplekit/canvas-mode";
setSKDrawCallback((gc) => { gc.fillRect(10, 10, 50, 50); });
startSimpleKit();
```
This will draw a simple rectangle for us, in the drawcallback is where we perform all drawing functions

can define colours with:
- Names `"red", "blue"`
- Hex `#0000ff`
- RGB `rgb(255, 0,0)`
- HSL `hsl(0deg 100% 100%)`

CanvasRenderingContext2D allows us to save and restore the state of drawing styles using `save()` and `restore()`

#### Drawable Objects
1. Define interface for an object that can be drawn:
```jsx
export interface Drawable { 
	draw: (gc: CanvasRenderingContext2D) => void; 
}
```
2. Define drawable objects like:
 ```jsx
 export class MyShape implements Drawable { ... 
	 draw(gc: CanvasRenderingContext2D) { // drawing commands go here } 
}
```
3. Create and draw:
```jsx
const myShape = new MyShape( ... ) 
myShape.draw(gc);
```

Painters Algorithm:
- go from back to front, with the item farthest back (eg horizon) being drawn first