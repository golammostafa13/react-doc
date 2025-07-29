# **React `useEffect` Good vs Bad Examples**

## **1. Initializing State (Bad vs Good)**
❌ **Bad:** Using `useEffect` for initial state
```jsx
const [data, setData] = useState(null);
useEffect(() => {
  setData(fetchInitialData()); // ❌ Unnecessary Effect
}, []);
```

✅ **Good:** Initialize state directly
```jsx
const [data, setData] = useState(fetchInitialData()); // ✅ Clean
```

---

## **2. Subscribing to Events (Bad vs Good)**
❌ **Bad:** Missing cleanup (memory leak risk)
```jsx
useEffect(() => {
  window.addEventListener('resize', handleResize); // ❌ No cleanup
}, []);
```

✅ **Good:** Proper cleanup
```jsx
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize); // ✅ Cleanup
}, []);
```

---

## **3. Fetching Data (Bad vs Good)**
❌ **Bad:** No cleanup (race condition risk)
```jsx
useEffect(() => {
  fetch('/api/data').then(res => setData(res.json())); // ❌ Risky
}, []);
```

✅ **Good:** Abort stale requests
```jsx
useEffect(() => {
  let ignore = false;
  fetch('/api/data')
    .then(res => res.json())
    .then(data => {
      if (!ignore) setData(data); // ✅ Safe
    });
  return () => { ignore = true; }; // Cleanup
}, []);
```

---

## **4. Updating State Based on Props (Bad vs Good)**
❌ **Bad:** Using `useEffect` to sync state with props
```jsx
const [internalValue, setInternalValue] = useState(props.value);
useEffect(() => {
  setInternalValue(props.value); // ❌ Causes extra renders
}, [props.value]);
```

✅ **Good:** Derive from props directly
```jsx
const internalValue = props.value; // ✅ No Effect needed
```

OR (if you need a mutable copy):
```jsx
const [internalValue, setInternalValue] = useState(props.value);
if (prevValue !== props.value) { // ✅ More efficient
  setInternalValue(props.value);
}
```

---

## **5. Running Logic on Mount (Bad vs Good)**
❌ **Bad:** Using `useEffect` for one-time logic
```jsx
useEffect(() => {
  trackPageView(); // ❌ Runs twice in StrictMode
}, []);
```

✅ **Good:** Use a ref to track execution
```jsx
const didLogRef = useRef(false);
useEffect(() => {
  if (!didLogRef.current) {
    didLogRef.current = true;
    trackPageView(); // ✅ Runs once
  }
}, []);
```

---

## **6. Debouncing/Side Effects (Bad vs Good)**
❌ **Bad:** Incorrect debounce in `useEffect`
```jsx
useEffect(() => {
  const timer = setTimeout(() => {
    search(query); // ❌ No cleanup, may leak
  }, 500);
}, [query]);
```

✅ **Good:** Proper cleanup
```jsx
useEffect(() => {
  const timer = setTimeout(() => {
    search(query); // ✅ Clean
  }, 500);
  return () => clearTimeout(timer); // Cleanup
}, [query]);
```

---

## **7. Syncing with External Stores (Bad vs Good)**
❌ **Bad:** Manual subscription
```jsx
useEffect(() => {
  const unsubscribe = store.subscribe(handleChange); // ❌ Manual
  return unsubscribe;
}, []);
```

✅ **Good:** Use `useSyncExternalStore` (React 18+)
```jsx
const state = useSyncExternalStore(store.subscribe, store.getState); // ✅ Built-in
```

---

## **8. Avoiding Infinite Loops (Bad vs Good)**
❌ **Bad:** State update in `useEffect` without dependency
```jsx
useEffect(() => {
  setCount(count + 1); // ❌ Infinite loop
}, [count]);
```

✅ **Good:** Use functional update if needed
```jsx
useEffect(() => {
  setCount(prev => prev + 1); // ✅ Safe
}, []); // No dependency
```

---

### **Key Takeaways**
- **Avoid `useEffect`** for derived state, prop synchronization, or event handling.
- **Use `useEffect`** only for side effects (API calls, subscriptions, timers).
- **Always clean up** subscriptions, timers, and async tasks.
- **Prefer built-in hooks** (`useMemo`, `useCallback`, `useSyncExternalStore`) when possible.
