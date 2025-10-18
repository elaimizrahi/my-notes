SVP, Engineering at Datadog

- Preventing circular dependencies
	- Don't want them since it becomes really hard to start up two nodes that depend on each other - how do you start one without the other?
	- Two options
		- 1. Ad-hoc breaking of dependencies to start one resource, then start the other, then fix dependencies
		- 2. Create a separate bootstrapper that allows one to start up without the other
- **Use AI to type, not to think**
- 