We must develop content to be as accessible as possible, no matter an indivual's physical and cognitive abilities and how they access a user interface (often abbreviated as **a11y**)
- It is wrong (and even illegal in many cases) to exclude someone from a website because they have a visual impairment
- It is a human right!

#### Modern OS Support for Accessibility
- Can control cursor from keyboard (motor)
- Adjust acceleration, precision, tracking
- Speech dictation
- Magnification
- Captions

#### Interface Enhancements

##### Visual Impairments
- zoom screen
- high contrast colours, dark mode, remove animations
- screen reader, voice input

##### Hearing Impairments
- Show audio visually
- Realtime audio proceessing to finter background noise
- monitor audio for certain sounds and send alert (baby crying)

##### Motor Impairments
- sticky, slow, and filter keys
- Reduce key repeat rate
- eye tracking
- voice input
- Brain computer interfaces

##### Cognitive Impairments
- word prediction, grammar, spell check
- text-to-speech
- augmenting text with icons/pictures

**The Curb-Cut effect** - Laws and programs designed to benefit vulnerable groups often end up benefiting all of society


### Implementation of Accessibility

##### Accessibility Tree
- browser generates an accessibility tree from the DOM with accessibility-related info for most HTML elements
	- name, role, description, state
- Devtools accessibility tab (**CMD-SHIFT-P**)

##### For Web Accessibility
- Use colour with care
- User ARIA roles and landmarks (when necessary)
- Design forms for accessibility

##### Semantic HTML
- don't use `<div>` and `<span>` for everything - name them properly and give clear sections for everything
	- instead use `<article>, <aside>, <footer>` etc.

##### ARIA Attributes
ARIA - accessible rich internet applications. It is a specification to add semantics to elements
- Use only if not possible to use an HTML semantic element (no equivalents for toolbar, feed, tooltip)
- ARIA states and properties should only be used if no HTML element equivalent (aria-required, aria-checked, aria-disabled)
- ARIA attributes only change the accessibility tree

##### Colour Perception
- Should account for colour blindness
- Harder to tell colours apart when
	- colours are pale
	- objects is small or thin
	- colour patches are far apart
- Contrast should be reduced - ie. light green on white
	- Follow WCAG colour contrast guidelines
		- **Minimum (AA) Level 4.5**
		- **Enhanced (AAA) Level 7.0**

##### Testing methods
- DevTools colour contrast tool
- Disconnect mouse and try to use app
- test with a screenreader
- A11y linters/checkers