Example:

```jsx
const [instance] = useState(()=> createInstance())
```

VS

```jsx
const instance = useRef(null)
React.useEffect(()=> instance.current = createInstance,[])
```

Pros:

1. It's concise, creating an instance in place rather than separating that process into an `effect`.
2. Because of #1, there is no need to check whether the instance is initialized.

All these benefits occur because `useRef` does not support providing value in a lazy (function) fashion.