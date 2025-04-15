The automatic update of the UI due to a change in the application's state

**VDOM Reconciliation**
VDOM is synced with the real DOM as follows:
1. Save copy of current VDOM
2. update VDOM with state or component changes
3. trigger re-render
4. compare VDOM difference
5. **Reconcile teh difference** by identifying a set of DOM patches
6. perform patch operations on real DOM
7. back to 1

**Changes**
- if a node **type** changes, the whole subtree is rebuilt
- if a node **type** is the same, attributes are compared and updated
- Use keys to make sure we can update children of the same node type (we know if we are altering or creating new ones - like adding to a list)

**Approaches**
1. UseState hook for local component states
2. useContext hook to access state without passing as props
3. signals to update models

**Hooks**
- hooks store data in a sequence of slopts associated with each component in the VDOM tree
	- calling a hook function uses up one slot and increments internal slot number counter so the next call uses the next slot
	- Preact resets this before invoking each component - so each hook is associated with the same slot
- This is called site ordering, and is why they must be called in the same order and not conditionally or in loops

- functional methods to compose state

**Signals** are a state management module introduced by Preact
- can be used with React and others
- acts as our model, but between different parts of our application 
	- constants
	- components
	- more!