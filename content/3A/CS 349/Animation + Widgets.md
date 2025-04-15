This is for [[SimpleKit]] animations and widgets, which is built off [[Javascript]]

- Key frames define the beginning and end of a tween
- Tweening is a process where we interpolate key frames into frames
- Easing is a function that controls how tweening is calculated
- Critical Clicker Frequency (CFF) is when our eyes start not noticing the flickering of changing between frames (usually 60-90Hz)
### Animation in simplekit
We can define a single animation callback in simplekit for updating animations (called with time in ms as an argument)
```jsx
type AnimationCallback = (time: number) => void; 
function setSKAnimationCallback(animate: AnimationCallback) ...
```

For example, if we want to animate a timer we want to input the current time, a duration, an update function, a method to check it's running, and a callback to call when done
```jsx
export class Animator { 
	constructor( public startValue: 
	number, public endValue: number, 
	public duration: number, 
	public updateValue: (p: number) => void ) {},
	public easing: EasingFunction = (t) => t
	...
```

We can create frames by using interpolation between keyframes using tweening:
![[Pasted image 20250225131745.png]]
And we can linear interpolation to tween:
`const lerp = (start: number, end: number, t: number) => start + (end - start) * t;`
Easing functions will make this interpolation smoother, for example: 
```jsx
const easeOut = (t) => Math.pow(t, 2); 
const easeIn = (t) => flip(easeOut(flip(t))); 
const easeInOut = (t) => lerp(easeOut(t), easeIn(t), t);
```

Some animation examples:
```jsx

export class BasicTimer {
	constructor(public duration: number) {}
	private startTime: number | undefined;
	start(time: number) {
		this.startTime = time;
		this._isRunning = true;
	}
	get isRunning() {
		return this._isRunning;
	}
	private _isRunning = false;
	update(time: number) {
		if (!this._isRunning || this.startTime === undefined) return;
		const elapsed = time - this.startTime;
		if (elapsed > this.duration) {
			this.startTime = undefined;
			this._isRunning = false;
		}
	}
}

let timeText = "";
setSKAnimationCallback((time) => {
	timer.update(time);
	timeText = `${(time / 1000).toFixed(1)}`;
});
setSKDrawCallback((gc) => {
	// clear canvas and draw it
	gc.clearRect(0, 0, gc.canvas.width, gc.canvas.height);
	gc.font = "24px sans-serif";
	gc.textBaseline = "top";
	gc.fillText(timeText, 10, 10);
	if (!timer.isRunning) dot.draw(gc);
});
const timer = new BasicTimer(3000);
timer.start(skTime);
startSimpleKit();
```


## Widgets
Smaller elements of the DOM, also called **components** or **elements** or **views**

We begin by creating an `AbstractWidget`, then using it to build our concrete widgets
- Could create an `AbstractButton` (like `SKButton`)
	- Then concrete classes may be `PlusButton`, `MinusButton`, `SubmitButton`
- We have boolean, number, choice, radio, and text widgets as some of the primitive ones for entering input
Used to display information and handle user input:
![[Pasted image 20250225134250.png]]

Button states can be manipulated by changing them on events, which we control with event handlers like `handleEvent(event: SKEvent)`
![[Pasted image 20250225134727.png]]

`WidgetContainers` can also be created, which may hold multiple widgets 
- form input with many text fields
- Tabs which select between which widget to draw

#### Example:
```jsx
export class SKButton extends SKElement {
	constructor({
	text = "?",
	width = 80,
	height = Style.widgetHeight,
	...otherElementProps
	}: SKButtonProps = {}) {
		super({ ...otherElementProps, width, height });
		this.text = text;
	}
	text: string;
	font = Style.font;
	state: "idle" | "hover" | "down" = "idle";
	draw(gc: CanvasRenderingContext2D) {
		gc.save();
			if (this.state == "hover" || this.state == "down") {
			gc.beginPath();
			gc.roundRect(this.x, this.y, this.width, this.height, 4);
			gc.strokeStyle = Style.highlightColour;
			gc.lineWidth = 8;
			gc.stroke();
		}
		gc.beginPath();
		gc.roundRect(this.x, this.y, this.width, this.height, 4);
		gc.fillStyle = this.state == "down"
			? Style.highlightColour
			: Style.defaultColour;
		gc.strokeStyle = "black";
		... // more drawing
		gc.restore();
	}
}
setSKDrawCallback((gc) => {
	gc.fillStyle = "white";
	gc.fillRect(0, 0, gc.canvas.width, gc.canvas.height);
	button.draw(gc);
});
startSimpleKit();
```


