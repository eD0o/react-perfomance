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
