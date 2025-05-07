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

## 2.2 - Pulling Content Up

