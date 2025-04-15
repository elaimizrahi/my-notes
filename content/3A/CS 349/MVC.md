
Separates data, state, and "business logic" from user-interface

![[Pasted image 20250225144252.png]]

1. Allows us to change view and controller implementation without changing model (ideally)
2. Supports multiple views of the same data (graph widgets)
3. Separation of code (view + controller, model are separate, easier for code reuse + testing)

**Model** handles app data and logic
**View** manages interface to present data
**Controller** manages interaction with data

We can use the observer pattern to build the MVC:
![[Pasted image 20250225144538.png]]
## MVC Pattern in TypeScript

### Observer

```typescript
export interface Observer {
  update(): void;
}
```

- **Observer Interface:** Defines the `update()` method that will be called when the model changes.

### Model

```typescript
export class Model extends Subject {
  // model data (i.e. model state)
  private _count = 0;

  get count() {
    return this._count;
  }

  // model "business logic"
  increment() {
    this._count++;
    // need to notify observers anytime the model changes
    this.notifyObservers();
  }
}
```

- **Model:** Stores the application state and contains business logic.
- **Increment Method:** Increases the `_count` and notifies observers when the state changes.

### View

```typescript
export class LeftView extends SKContainer implements Observer {
  update(): void {
    this.button.text = `${this.model.count}`;
  }

  button: SKButton = new SKButton({ text: "?" });

  constructor(private model: Model, controller: LeftController) {
    super();
    this.addChild(this.button);

    // Set an event handler for the button "action" event
    this.button.addEventListener("action", () => {
      controller.handleButtonPress();
    });

    // Register with the model when ready
    this.model.addObserver(this);
  }
}
```

- **References to model and controller:** The `LeftView` references both `model` and `controller` through its constructor.
- **Connect to controller:** The button's event handler calls `controller.handleButtonPress()`.
- **When model changes:** The `update()` method updates the button text whenever the model changes.

### Controller

```typescript
export class LeftController {
  constructor(private model: Model) {}

  handleButtonPress() {
    this.model.increment();
  }
}
```

- **References to model:** The controller holds a reference to the `model` and calls its `increment()` method when the button is pressed.

Usually we **tightly couple** the view and controller, so the view will integrate the controller within itself

### View with Integrated Controller

```typescript
export class LeftView extends SKContainer implements Observer {
  update(): void {
    this.button.text = `${this.model.count}`;
  }

  button: SKButton = new SKButton({ text: "?" });

  constructor(private model: Model) {
    super();
    this.addChild(this.button);

    // Controller integrated directly within the view
    this.button.addEventListener("action", () => {
      this.model.increment();
    });

    // Register with the model when ready
    this.model.addObserver(this);
  }
}
```

- **Integrated Controller:** The view directly handles model updates without a separate controller.
    
- **When model changes:** The `update()` method updates the button text whenever the model changes

## Other MV* Implementations

![[Pasted image 20250225150327.png]]
![[Pasted image 20250225150335.png]]
![[Pasted image 20250225150341.png]]
![[Pasted image 20250225150345.png]]![[Pasted image 20250225150345 1.png]]

