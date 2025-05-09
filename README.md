# 2 - Reducing Renders

## 2.1 - Push State Down

Lift state only as much as necessary.
`Keeping state too high in the tree leads to unnecessary rerenders` of components that don't actually depend on that state.

### 🤯 What Triggers a Rerender?

A component function runs again when:

- Its state changes
- Its props change
- Its context changes

> ⚠️ This doesn't mean the DOM always changes — React does a reconciliation step to decide what really updates.

### 📉 The Problem With High-Up State

If you store state in a parent or globally:

- `Any change causes all its children to rerender, even those not using the updated state`.
- Performance suffers in large trees.

### ✅ The Solution: Push State Down

`Move state closer to the leaf component` that uses it.

#### Before (state too high):

```tsx
function Form() {
  const [name, setName] = useState("");
  return <Input value={name} onChange={setName} />;
}
```

#### After (state pushed down):

```tsx
function Input() {
  const [value, setValue] = useState("");
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

Now only Input rerenders when value changes, not the whole Form.

### Use Case: Large Forms

If a form has many inputs:

- Avoid putting all input values in one shared state.
- Instead, let each input manage its own local state.
- Then, lift all values once on submit, or when you need coordination (e.g., live validation).

### When to Lift State

Move state to parent only if:

- You need to coordinate between components (e.g., validation, submission).
- A parent needs to control or respond to changes.

### Summary Table

| Scenario                          | Use Local State | Lift State Up                                               |
| --------------------------------- | --------------- | ----------------------------------------------------------- |
| ✅ Single input field             | Yes             | No                                                          |
| ✅ Independent form inputs        | Yes             | No                                                          |
| 📋 Form submission                | No (on submit)  | Yes (collect all inputs)                                    |
| 🧪 Field validation (isolated)    | Yes             | No                                                          |
| 🧪 Field validation (cross-field) | No              | Yes                                                         |
| 📦 Cart item quantity             | Yes             | Yes (lift if total price or stock availability must update) |
| 📅 Date range picker              | No (needs sync) | Yes                                                         |
| 📚 Tabs (switching view)          | No              | Yes (parent controls active tab)                            |
| 📂 Accordion (open/close logic)   | No              | Yes (if only one section should be open)                    |
| 🖼️ Gallery (select thumbnail)     | No              | Yes (parent updates selected image)                         |

---

## 2.2 - Reducing Rerenders: Pull Content Up

When a component doesn't depend on the state or props of its parent, you can "pull it up" `(move it higher in the tree) to avoid unnecessary rerenders`.

### 🎯 The Problem: Nested Expensive Components

`If you nest a heavy component like ExpensiveComponent inside a parent that rerenders often, React will re-run the child unnecessarily` — even if its inputs didn’t change.

> ⚠️ React will not skip rendering a child unless its position in the tree is stable and memoized.

### ✅ The Solution: Pull Content Up

Move stable components outside of frequently rerendered parents — even if that means passing them as children.

#### Before (heavy component rerenders too often):

```tsx
const Game = () => {
  return (
    <>
      {/* State-driven rendering */}
      <GameInput />
      <ExpensiveComponent /> // rerenders every time Game rerenders
    </>
  );
};
```

#### After (stable component hoisted):

```tsx
const Application = () => {
  return (
    <Game>
      <ExpensiveComponent /> // passed as children, never redefined
    </Game>
  );
};
```

```tsx
const Game = ({ children }) => {
  return (
    <>
      <GameInput ... />
      {children} // doesn't rerender unless children change
    </>
  );
};
```

Now ExpensiveComponent is `only rendered once unless its own props change`.

### 💡 Why It Works

- React compares children by reference.
- Pulling stable components up ensures they’re not redefined on every render.
- This is `especially helpful for expensive or memoized components`.

### 🧠 When to Pull Content Up

Use this strategy if:

- A child component is `expensive to render`
- It `doesn’t depend on dynamic state/props from the parent`
- It `could be memoized` (React.memo) or kept stable

### Summary Table

| Scenario                               | Push State Down | Pull Content Up       |
| -------------------------------------- | --------------- | --------------------- |
| ✅ Local input control                 | Yes             | No                    |
| 🧠 Stable, heavy child component       | No              | Yes                   |
| 📦 Reusable UI with no state coupling  | No              | Yes                   |
| 🧪 Components depending on local state | Yes             | No                    |
| ⚙️ Performance bottlenecks in tree     | Maybe           | Yes (hoist stateless) |

---

## 2.3 When to Use React.memo, useMemo, and useCallback

These hooks and utilities `help optimize rendering performance by avoiding unnecessary recalculations or re-renders`.

> However, overusing them can harm readability and even performance. Use them strategically when performance is a real concern.

### 📊 Comparison Table

| Feature     | Purpose                           | Use When…                                                             | Returns               | Common Pitfall                           |
| ----------- | --------------------------------- | --------------------------------------------------------------------- | --------------------- | ---------------------------------------- |
| React.memo  | Prevents re-renders of components | A component receives the same props across renders                    | A memoized component  | Won’t work if props change every time    |
| useMemo     | Memoizes a value or computation   | A calculation is expensive or a new object causes unwanted re-renders | The memoized value    | Overused for cheap operations            |
| useCallback | Memoizes a function               | A callback is passed to memoized children                             | The memoized function | Use only when function reference matters |

### React.memo example: Avoid re-render when props are the same

```tsx
const UserName = React.memo(({ name }: { name: string }) => {
  console.log("Rendering UserName");
  return <div>{name}</div>;
});

export const App = () => {
  const [count, setCount] = useState(0);
  return (
    <>
      <UserName name="Du" />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </>
  );
};
```

Use React.memo when a component:

- `Is pure` (output only depends on props)
- Receives `props that change infrequently`
- Avoids re-rendering when its parent re-renders unnecessarily

### useMemo example: Memoize an expensive computation

```tsx
export const App = () => {
  const [num, setNum] = useState(1);

  const factorial = useMemo(() => {
    console.log("Calculating factorial");
    return Array.from({ length: num }, (_, i) => i + 1).reduce(
      (a, b) => a * b,
      1
    );
  }, [num]);

  return (
    <>
      <p>Factorial: {factorial}</p>
      <button onClick={() => setNum((n) => n + 1)}>Increment</button>
    </>
  );
};
```

Use useMemo when:

- You have `expensive calculations`
- You’re `returning objects/arrays used in dependencies`
- Avoiding unnecessary changes in reference equality is crucial

### useCallback example: Keep function identity stable for memoized children

```tsx
const Button = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log("Rendering Button");
  return <button onClick={onClick}>Click me</button>;
});

export const App = () => {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("Clicked!");
  }, []); // stable reference

  return (
    <>
      <Button onClick={handleClick} />
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </>
  );
};
```

Use useCallback when:

- You're passing `callback props to memoized children` (React.memo)
- You need to `maintain stable function references`

Yes — here are a few important final insights to round out the section and help you apply these tools more wisely:

### 🧠 Final Tips on Performance Hooks

#### Only Optimize When Needed

- `Measure before optimizing`. Premature optimization can lead to bloated code.
- Use tools like `React DevTools Profiler, why-did-you-render, or Performance tab in DevTools` to find real bottlenecks.

#### 📎 Dependencies Matter

- `useMemo and useCallback only work correctly if their dependency arrays are accurate`.
- `Missing a dependency can lead to bugs` that are hard to trace.

#### 🧩 React.memo + useCallback ≠ Always Better

- Even if you memoize a component and its props, `React still needs to do shallow comparison`.
- If props are complex or change often, the cost of memoization can outweigh the benefits.

#### 📦 Consider Context Scope Too

- If you're using a global context or a large parent state, breaking up components and pushing state downward can often improve performance more than hooks.

### 🚫 Anti-pattern Example (Overuse)

```tsx
// Overkill: memoizing everything for a cheap operation
const ExpensiveCalculation = React.memo(({ value }: { value: number }) => {
  const result = useMemo(() => value * 2, [value]);
  return <div>{result}</div>;
});
```

> ⚠️ This adds unnecessary complexity. Multiplying by 2 isn’t "expensive". Don’t optimize what’s not a problem.

### 🧰 Bonus Tool: `why-did-you-render`

If you're optimizing render behavior, consider installing:

```bash
npm install @welldone-software/why-did-you-render
```

Then:

```ts
import React from "react";
if (process.env.NODE_ENV === "development") {
  const whyDidYouRender = require("@welldone-software/why-did-you-render");
  whyDidYouRender(React);
}
```

It tells you why a component re-rendered — great for understanding which props or contexts changed.

---

## 2.4 - useReducer & dispatch

The useReducer hook is a powerful alternative to useState, `especially useful when managing complex state logic or coordinating updates across deeply nested components`. It shines when you need to:

- Update state based on previous state
- Centralize logic around how state updates happen
- Avoid unnecessary useCallback usage by stabilizing handlers

### 🧠 Key Concepts

- useReducer returns a state and a dispatch function.
- You send actions (usually objects) to the reducer via dispatch.
- The reducer contains all update logic, acting as a central authority.

```tsx
const [state, dispatch] = useReducer(reducerFn, initialState);
```

### ✅ Benefits Over useState with useCallback

| With useState                                                   | With useReducer                                        |
| --------------------------------------------------------------- | ------------------------------------------------------ |
| Need to define update, remove, etc. with useCallback            | `dispatch is static, no need for memoized callbacks`   |
| Can trigger re-renders by prop changes due to unstable handlers | dispatch stays the same across renders                 |
| Manually juggle logic in event handlers                         | Centralize logic in reducer: easier to test & maintain |

### Example: Toggling Packed State

Using useState + useCallback:

```tsx
const [items, setItems] = useState(initialItems);

const togglePacked = useCallback((id) => {
  setItems((items) =>
    items.map((item) =>
      item.id === id ? { ...item, packed: !item.packed } : item
    )
  );
}, []);
```

Using useReducer:

```tsx
function reducer(state, action) {
  switch (action.type) {
    case "TOGGLE_PACKED":
      return state.map((item) =>
        item.id === action.id ? { ...item, packed: !item.packed } : item
      );
    default:
      return state;
  }
}

const [items, dispatch] = useReducer(reducer, initialItems);

// Use it like:
dispatch({ type: "TOGGLE_PACKED", id: itemId });
```

Because dispatch is always the same reference, passing it down avoids re-renders that occur when update or remove functions are re-created via useCallback. You can remove many useCallback calls just by centralizing the logic in the reducer.

This makes useReducer a performance win, especially in memoized lists.

### 🧪 Testing Advantage

With a pure reducer function, logic becomes trivially testable:

```ts
it("toggles packed", () => {
  const state = [{ id: 1, packed: false }];
  const newState = reducer(state, { type: "TOGGLE_PACKED", id: 1 });
  expect(newState[0].packed).toBe(true);
});
```

### 🧭 When to Prefer useReducer

Use useReducer when:

- State transitions are non-trivial or interrelated
- You’re building a form with many fields
- You want to decouple state update logic from component hierarchy
- You want better testing and readability for state changes

---