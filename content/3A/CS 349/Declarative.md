Imperative programming describes *how* to achieve a result, but declarative programming says *what* result we want

[[SimpleKit]] used imperative programming - we had to build the tree of nodes with events. We described how to make the tree

[[HTML + CSS]] can be declarative. We say what we want the end result to look like

**HyperScript** is a fully declarative language used to generate descriptions of UI trees. It's an `npm` package to *generate* HyperScript
- Hyperscript function calls create a representation of the UI tree as a JS object
	- commonly referred to as a virtual DOM or a vdom
- Used to render an actual DOM with imperative methods
- Is a lightweight abstraction of DOM so that changes can be compared (for DOM diffing)

**JSX** is syntactic sugar for HyperScript. It is compiled into JavaScript and HyperScript, where HyperScript is used to render.

### Preact
Goal is to render quickly and efficiently while being lightweight. Aims to be largely compatible with React
- Uses native DOM events
- Treats children nodes as native JS arrays 
- uses `class=` to set class attribute (not `className`)

Note: files wiwth JSX are compiled into JavaScript with HyperScript function calls

**Components** build the preact application - in the same way as react
- can be defined as functional components (most common) 
- Can also be defined as class components - not common and functional ones are better

**children** array is a special implicit prop that can be accessed in preact

**Typescript** requires type definition for component props

**Must** return exactly one root node in the tree per function

**Dynamic attributes** can be set by using template literals `${colour}` or hyperscript object `backgroundColor: color` <- note that camelCase is used here

