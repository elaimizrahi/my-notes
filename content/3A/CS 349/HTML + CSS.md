A quick rundown of HTML and CSS for the CS 349 midterm
### HTML

- **HTML**: Hypertext Markup Language that defines the meaning and structure of web content.
- Uses **declarative syntax** (vs. imperative syntax).

#### Structure

- **Basic Document:**
    - `<DOCTYPE html>`, `<html>` (root)
    - `<head>`: Includes `<title>` and meta elements
    - `<body>`: Contains page content

#### Tags, Attributes, Elements

- **Tag:** Defines elements and attributes: `<tag>content</tag>` or `<tag />`
- **Attribute:** Provides extra info: `<input type="text" />`
- **Element:** Resulting object created by tags/attributes

#### Document Object Model (DOM)

- Treats HTML as a **tree of nodes**
- Node Types: Document, Element, Attribute, Text, Comment

#### Common Tags

- **Containers:** `<div>` (block-level), `<span>` (inline)
- **Widgets:** `<button>`, `<input>`, `<label>`, `<textarea>`, `<select>`

### CSS

- **CSS:** Cascading Style Sheets for styling and layout
- Uses **declarative syntax**

#### CSS Rules

- **Structure:** Selector → Declaration Block → Properties
- Example:

```css
div > #box {
  background-color: blue;
  padding: 10px;
}
```

#### Selectors

- Basic: `div`, `.class`, `#id`, `[type="text"]`
- Hierarchical: `div > div`, `div div`
- Pseudo-class: `:first-child`, `:hover`
- Combine: `div#a`, `div, #a`

#### The Cascade

- **Order of Precedence:** Browser → Author → User
- **Sources:** External file → `<style>` tag → Inline style
- **Specificity:** More specific selectors override less specific ones
- **Important:** `!important` overrides other rules

### Flexbox Layout

- **Container:** `display: flex;` aligns children as flex items
- **Axes:** Main (row/column) and Cross (perpendicular)
- **Item Properties:**
    - `flex-grow`: Grow proportionally
    - `flex-shrink`: Shrink proportionally
    - `flex-basis`: Initial size (auto/content/number)
- **Alignment:**
    - Container: `justify-content` (main axis), `align-items` (cross axis)
    - Items: `align-self` (cross axis)

### DOM Manipulation with TypeScript

- **Selecting Elements:**
    - By ID: `document.getElementById("id")`
    - By Selector: `querySelector("#id")`
- **Imperative Approach:**
    - `createElement`, `appendChild`, `innerText`
- **Declarative Approach:**
    - `innerHTML`, `insertAdjacentHTML`

### Events

- Use `addEventListener()` for events like `click`
- Event phases: Capture, Target, Bubble
- Prevent propagation: `event.stopPropagation()`

### Resources

- [HTML Basics](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics)
- [CSS Basics](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/CSS_basics)
- [DOM Manipulation](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents)