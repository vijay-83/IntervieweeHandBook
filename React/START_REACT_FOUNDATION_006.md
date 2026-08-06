
# START_REACT_FOUNDATION_006

# React Forms, Events & UI Patterns (Q41–Q50)

## Q41. Controlled Components
React manages the input value through state.

```jsx
const [name,setName]=useState("");
<input value={name} onChange={e=>setName(e.target.value)} />
```

## Q42. Uncontrolled Components
The DOM stores the value. Access it using `useRef`.

## Q43. Controlled vs Uncontrolled

| Controlled | Uncontrolled |
|---|---|
| React state | DOM state |
| Easier validation | Less code |
| Predictable | Better for simple forms |

## Q44. Event Handling
```jsx
<button onClick={handleClick}>Save</button>
```

## Q45. What are Synthetic Events?
A React wrapper that provides consistent browser event behavior.

## Q46. Rendering Lists
```jsx
items.map(item => <li key={item.id}>{item.name}</li>)
```

## Q47. Why are Keys Important?
Keys help React identify items efficiently during reconciliation. Use stable unique IDs.

## Q48. Conditional Rendering
- if/else
- Ternary (`condition ? A : B`)
- Logical AND (`condition && <Comp/>`)
- Early return

## Q49. What are Fragments?
Fragments group multiple elements without adding extra DOM nodes.

```jsx
<>
  <Header/>
  <Content/>
</>
```

## Q50. What are Error Boundaries?
Error Boundaries catch rendering errors in child components and display fallback UI. Traditionally implemented with class components or framework wrappers.

### Best Practices
- Validate form inputs
- Never use array index as key when list order can change
- Keep forms controlled for complex applications
- Extract reusable form components
- Handle loading and error states consistently
