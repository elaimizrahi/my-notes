**Checkpointing** - a manual undo method where you save the current state so you can go back to it later

Some actions may be omitted from undo since they can't be easily undone - like deleting a file

**All** changes to a document should be undoable - these are stored in the model
**No** view changes to view (interface) should be undoable **Unless** it's tedious or requires significant effort (make it easy for the user) 
**Always** ask for confirmation before doing a destructive action that is undoable

**State Restoration** should ensure that the user interface should remain how it was before (highlighted, underlined, etc)

**Granularity** should be considered - how do we **chunk** the actions that can be undone
 - should an undo remove an entire paragraph of text or the last word?

#### Implementing Undo
- **Forward Undo** start from base, the maintain list of changes to compute current document
	- Undo by removing last change from list
- **Reverse Undo** apply change to update document, but also save reverse change
	- undo by applying reverse change to document (think db migrations)
- **Change record** defines a single transformation to the document

- **With Stacks** 
	- use either an **Undo Stack** to keep all change record, saved as you do actions
	- or an **Redo Stack** with change records that have been undone

- **Reverse change record**
	- Can use **Command** pattern where it saves command and reverse command to reverse state
	- Or **Memento** pattern where it saves snapshots of each doc state, either the entire thing or just the difference

- **Destructive commands** 
	- Can use forward command undo
	- Or use reverse command undo, but un-execute command stores previous state for destructive commands - might require a lot of memory


