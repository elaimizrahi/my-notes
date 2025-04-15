
# Dispatch

use a central `handleEvent(e: SKEvent)` function to handle all events
- based on the event, we can call `handleEvent` in our widgets to let them handle each event internally
UI toolkits organize widgets into a tree, with one root element
- ordered by draw order of left to right (top to bottom in code)
- for non-leaf nodes, we need `containers` to hold sub-widgets which will be leaf nodes (or sub-containers)

Event pipeline:
![[Pasted image 20250225135914.png]]
![[Pasted image 20250225135925.png]]

#### Steps of Event Dispatch
1. Target Selection
	- frontmost widget under mouse
2. Route construction
	- path from root to target
3. Propogation (Event Binding)
	- capture **down** from root to target
	- bubble **up** from target to root
**Note: ** Route construction and propagation only apply to positional dispatch (mouse event only)
- Key events are **Focus Dispatch**
- Mouse eventts are **Positional Dispatch**
![[Pasted image 20250225140923.png]]
**Observe that we use [[Input + Hit testing]] to see if the child is in the route**

#### Event Binding
How we associate events with code (route, send to code, handle with code...)

Early windows used global event callbacks, similar to what [[SimpleKit]] uses for `setSKEventHandler`

Drawbacks:
- Difficult to maintain
- Events are not delegated to an object
- better if widgets handle events themselves (modularity)


Instead we can use **Inheritance Binding**
- An event is dispatched to a widget base object, which concrete classes will handle implementation of
- Still has issue where multiple event types are processed through each event method, but we can be more specific with switch cases (not always checking `key event` on non-text input widget)
- Doesn't scale well and can cause performance issues

We can also use **Listener Binding**:
- The event listener is where each type of event is handled (ie `MouseMoved`) and widgets call the event listener accordingly

[[SimpleKit]] uses a form of inheritance binding with listener objects
- Each `SKElement` binds events to event handlers
- toolkit events like `mousemove` and `mousedown`
- widget events like `action` when a button is clicked

It maintains a table of `BindingRoutes` which is then used to map event types to their handlers, which we add to by calling `addEventListener(type: string, handler: EventHandler)`

In our widgets, we can also create methods to override SimpleKit's defaults
- `handleMouseEvent(me: SKMouseEvent)` for example

We can propagate by either bubbling up or capturing down, with the option to stop propagation:![[Pasted image 20250225142407.png]]

- Sometimes we want to stop propagation to prevent some events from bubbling up to their default events (e.g. click on icon selects it, click on background deselects all icons)
#### Focus Dispatch
- Events are dispatched to a widget regardless of mouse cursor position.
- Needed for all keyboard and some mouse events:
    - **Keyboard focus:** Clicking on a text field, moving the cursor off, and starting to type.
    - **Mouse focus:** Mouse down on a button, moving off, and mouse up (also called “mouse capture”).
- Maximum one keyboard focus and one mouse focus at a time.
    - **Why?** Ensures consistent input behavior and prevents conflicting interactions.
- Widgets need to gain and lose focus at appropriate times.

**Note: Focus Dispatch Needs Positional Dispatch**
- A `mousedown` event sets mouse and keyboard focus to a widget.
    - Only text entry widgets should request keyboard focus.
    - Any widget can request mouse focus.
- Other ways to request focus:
    - **TAB key:** Navigate a UI without a mouse (assumes a defined “tab order” for widgets).
    - **App-initiated:** An app can request focus itself (e.g., pressing ENTER moves keyboard focus to the next text field).

### Keyboard Focus Dispatch
- Keyboard dispatch uses `focusedElement`.
- `requestKeyboardFocus`: Allows elements to request keyboard focus.
    - Used in `SKTextfield` within `handleMouseEvent`.
- "focusout" events are created and immediately dispatched to the widget’s `handleKeyboardEvent`.
    

### Mouse Focus Dispatch
- Mouse focus is handled in the `mouseDispatch` function.
- `focusedElement` is a module variable.
- `requestMouseFocus`: Allows elements to request mouse focus.
    - Used by `SKButton` within `handleMouseEvent`.
        

# Layout
The visual arrangement of widgets in the UI


For [[SimpleKit]], it uses only 3 dimensions: margin, padding, and content(width, height) (top right bottom and left cannot be set separately)

- **Content Size** is the size of what's inside the widget
- **Intrinsic Size** is the normal widget size
- **Layout Size** is the amount of space the widget occupies in the layout (with padding + margin)

Simplekit implements the box model uses `SKContainer` elements, where `measure()` gets the intrinsic size, `layout` sets the element layout size

By calling `.layout()` for the tree root, it will set the position and size of all widgets 

**Fixed Layout** allows elements to set they x, y, width, height
- simplest layout algorithm
- called with `root.layoutMethod = new Layout.FixedLayout()`

**Centred Layout** allows elements to be centred in their containers
- centering calculation using layout bounds
- called with `root.layoutMethod = new Layout.FixedLayout()`

**Wrap Layout** allows elements to be wrapped into rows
- row height is the largest element height in the row
- called with `root.layoutMethod = new Layout.WrapRowLayout({gap: 10})`
- gap is the space between rows

**Responsive Layout** allows elements to dynamically move, reposition, resize, and hide content
- based on change in size of application window

**Basis Sizes** are predefined sizes for width and height for widgets, which can be overridden when being constructed

**Fill Layout** allows elements to fill the width of their parent container
- row height is the largest element height in the row
- called with `root.layoutMethod = new Layout.FillRowLayout({gap: 10})`
- gap is the space between rows

**Every time widget size or layout properties change, at least some of the UI must be re-rendered. The best practice is to run layout at most once per run-loop frame**



