## Effects
An effect is when a component function is called for rendering

**Side effects** are changes outside the Vdom render. Code is run as a result of the Vdom change
- Examples
	- Logging
	- DOM manupulation
	- fetching data
	- drawing
- Hooks
	- useEffect
	- useMemo
	- useCallback
	- useRef

**HTML Form Element** values are associated with application state - we have to manage component data flow
- **Uncontrolled**
	-  Form elements manage their own data values
	 ![[Pasted image 20250408160055.png]]
- **Controlled**
	- Application state manages HTML Form Element data values
	- ![[Pasted image 20250408160202.png]]


## Styling
**Approaches** 
- Separate CSS file for each component
- Inline CSS
- Import component specific CSS
- Utility classes (tailwind)

**Inline CSS** - style attribute accepts JS object to set CSS properties
- do not provide flexibility with rules, selectors, etc.
- written in camel case

**CSS Modules** - CSS file with `.module.css` extension 
- import module.css into the component tsx file
- Class names are **Locally Scoped** to the component 

**Tailwind** - a utility-first framework that works best with declarative programming (Preact)
- No need for media queries, use a size prefix instead


