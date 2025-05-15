# 3 - Context

## 3.1 - Context API

Context API `solves prop-drilling by allowing data to be passed deeply through the component tree` without explicitly passing props at every level. `However, context updates bypass some memoization optimizations, triggering re-renders in all consuming components, even if useMemo or React.memo are used`. This highlights the trade-off between convenience and performance when using Context.

### Problem (packing-list repo)

- dispatch or setItems had to be passed through intermediate components that didn't use them.
- While this didn’t harm performance (thanks to stable references), it made the code messy and harder to scale.

### Solution: Context API

- createContext() defines a `shared value accessible to any component using useContext()`.
- A Provider component wraps part of the app to expose shared values (e.g. items, dispatch).
- Simplifies component trees by removing the need to pass props like dispatch or items.

![](https://i.imgur.com/LJAb1qu.png)

### Refactor Steps

1. Create context with createContext().
2. Define a Provider component (ItemsProvider) that holds state and reducer.
3. Wrap the app in this provider.
4. Use useContext in deeply nested components to access values directly.
5. Delete redundant prop-passing.

### Trade-offs / Gotchas

- Switching to context caused all consuming components to re-render, even though memoization was in place.
- Reason: Context is a hidden prop, when it changes, all consumers re-render.
- useMemo doesn’t prevent re-renders triggered by context updates.

### Core Insight

> "Context avoids prop-drilling but behaves like a prop. If its value changes, all subscribers re-render."

### Performance Tip

- Use context wisely, `Avoid storing frequently changing values (e.g., large lists, local state) in context, as it causes unnecessary re-renders across all consumers.`.
- Consider splitting context or `combining it with techniques like useMemo, useCallback, or selector patterns`.

## 🧠 3.2 - Using Multiple Contexts

### 🧩 The Core Problem

A single object combining multiple values ({ items, dispatch }) passed to context leads to unnecessary re-renders:

- Even if items and dispatch haven't changed, the object reference is new each render.
- React.memo comparisons break because of the new object.
- This results in memoized components rerendering needlessly.

### 🔍 React Rules Revisited

- Even when individual values (like dispatch) remain unchanged, the `context is treated as changed due to new object identity`.
- React's reactivity is working correctly, but it's `causing unintended side effects in this case`.

### 🛠️ The Solution: Use Two Contexts

> One shared context can lead to unnecessary re-renders; splitting it into multiple contexts allows more granular subscriptions..

### ✅ Why Two Contexts?

- Split the context into:

  - ItemsContext – for values that change frequently (items)
  - ActionsContext – for values that rarely change (dispatch)

- `Components only re-render when the part they care about changes`.

### ⚠️ Order Matters

- Place static context (dispatch) outside the dynamic one (items) to `avoid unnecessary child re-renders`.
- Parent context triggers children rerenders — so order affects performance.

```tsx
<ActionsContext.Provider value={dispatch}>
  <ItemsContext.Provider value={items}>{children}</ItemsContext.Provider>
</ActionsContext.Provider>
```

> You can nest multiple Providers, but evaluate whether the added complexity is justified by the performance benefits.

```tsx
return (
  <ActionsContext.Provider value={actions}>
    <UsersContext.Provider value={users}>
      <PostsContext.Provider value={posts}>{children}</PostsContext.Provider>
    </UsersContext.Provider>
  </ActionsContext.Provider>
);
```

```tsx
export const useActions = () => {
  return useContext(ActionsContext);
};

export const useUsers = () => {
  return useContext(UsersContext);
};

export const usePosts = () => {
  return useContext(PostsContext);
};
```

### 🧼 Abstraction: Hiding the Complexity

#### 🧱 Create Custom Hooks

```tsx
const useItems = () => useContext(ItemsContext);
const useDispatch = () => useContext(ActionsContext);
```

- Encapsulates “bad” complexity.
- Simplifies switching to Redux or other state libraries in the future.
- Mirrors React-Redux patterns for familiarity (useDispatch etc.).

> Abstracting complexity is fine, as long as it's well-documented and easy to refactor later (e.g., migrating to Redux).