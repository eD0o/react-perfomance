# 1 - React Rendering

## 1.1 - Inspect Performance with Dev Tools

### 🚫 Don't Skip These Basics

Before digging into performance optimizations, make sure:

- App is running in Production Mode

  - `Development mode adds overhead`: console logs, dev warnings, extra React DevTools features, etc.
  - You might misjudge performance if you profile a dev build.
  - If you see a red React icon 🔴 on your production site (via React DevTools), that may be that you're in dev mode.
  - Fix: set the Webpack environment to production. This is usually handled automatically by tools like create-react-app, but custom setups may require manual fixes.

- Use proper keys in lists

  - `Always provide a unique and stable key when mapping over a list of elements` in React.
  - `Avoid using array indices as keys`, they shift with sorting or changes, causing unnecessary re-renders or bugs.
  - React will warn you in the console if you mess this up. Pay attention to the warning.

### 🔧 Install React Developer Tools

- Available for Chrome, Edge (via Chrome extensions), Firefox, etc.
- Adds two tabs in browser DevTools:

  - Components tab: `view component tree, props, and state`. You can inspect and tweak them, great for debugging but not performance-focused.
  - Profiler tab: `main tool for analyzing rendering performance`.

### 🎥 How to Use the Profiler

- Press Record and perform some interactions (e.g., removing a user, adding a post).
- Review each commit to see:

  - What rendered
  - Why it rendered
  - Parent/child relationships

![](https://i.imgur.com/F9rybFB.png)

> ⚠️ Just because something re-renders doesn’t mean it's a problem.
> Rendering itself is not always a performance issue, focus on symptoms first.

### ✅ When to Start Profiling

- You notice slowness or get reports like "it’s slow."
- Classic example: typing in an input feels laggy or characters appear all at once.
- Another example: full browser lock-ups.
- Start with a known issue, then profile and investigate, don’t randomly optimize.

`Measure first before you optimize, then measure again`.

### 💡 Visual Clues for Re-renders

- `Enable “Highlight updates when components render” in React DevTools`.

![](https://i.imgur.com/42EN5iz.png)
![](https://i.imgur.com/HUXZ8i4.png)

- `You’ll see green boxes flash when components update`.
- Helps spot excessive re-renders in real time.
- Only visible when DevTools are open.

> Be mindful: closing DevTools may make your app look faster. Don’t be fooled.

### 🧪 Simulate Real-World Conditions

- Use the built-in tools to `throttle network and CPU`:

  - Especially useful if you're testing on a high-end machine.
  - Emulates average user environments (e.g., slow CPU + poor Wi-Fi).
  - Great for understanding how your app performs in realistic conditions.

Developers often have better hardware than users. Simulate slowness to catch real-world issues.

### 🧠 Takeaways

- Ensure production mode is enabled
- Always use proper React keys in lists
- use Profiler tab to diagnose actual re-render causes
- Throttle to test for performance under constrained conditions
- Don’t blindly optimize — find a problem first, then fix it

## 1.2 - Beyond React Profiler: When and Why to Use Chrome DevTools

### 🔍 DevTools are Still Essential

- React Profiler is helpful but not always sufficient.
- Chrome DevTools (console, debugger, breakpoints, elements tab) remain critical for general JavaScript debugging and DOM inspection.
- `These tools work well in parallel`. You might be looking at green render flashes while also checking console logs or breakpoints.

### 🕹️ React Profiler Overhead

- Profiling introduces overhead and slows down the app:

  - `Runtime could be 2–3x slower while profiling`
  - The actual values are inflated, don’t assume they reflect production speed

- Use relative comparisons — for example, if a commit took 12% longer than last sprint, that’s still a red flag
- To get closer to real performance:

  - `Use production profiling builds` (custom-configured for tools like Vite, Webpack, CRA)
  - These keep profiling enabled but without full development overhead

  > Note: you might lose some helpful debug info like component names

### 📐 Relative Numbers Matter More Than Raw Metrics

- Focus on whether performance improved or worsened,, not just absolute numbers
- Don’t get distracted by exact milliseconds
- Use trends to detect regressions or improvements

### ⚠️ Over-Optimizing Can Be a Trap

- Memoization, render prevention, etc., are only useful if the cost of skipping is less than the cost of doing
- `Sometimes memoizing or preventing a render costs more than just rendering`
- `Always measure before optimizing`

### 🪄 Steve’s Golden Rule of Performance

`Anything you don’t do is faster`

- This applies to `avoiding unnecessary renders, memoizations, or premature loading`
- React 18 lets you delay non-urgent work to prioritize user interactions (like typing or clicking)
- Avoid locking up the UI for background tasks

### Takeaway:

- Be cautious of `immediately-invoked functions in useState`.
- `Use lazy initialization for expensive operations`, eg: `useState(() => generateRandomColor())`
  - This delays execution until React calls it, avoiding pre-emptive JavaScript evaluation and unnecessary re-renders.
- `Measure before fixing`, profiling tools help identify and validate real bottlenecks.

### 🧠 Final Thoughts

- Performance is about doing less, not thinking harder
- `Preventing unnecessary work is better than optimizing necessary work`
- Only optimize when there's a measurable problem
- Don’t trust blogs blindly use actual performance data

---

## 1.3 - React Rendering Cycle

Understanding React’s rendering cycle is essential to diagnosing performance issues. `React doesn’t rerender “just because” something has to trigger it`.

### 🚦 What Triggers a Render?

- User interaction (clicks, typing, navigation)
- State updates (useState, useReducer)
- Prop changes
- Context updates
- Polling intervals or subscriptions

Each of these can cause a component (and its children) to rerender. While React is efficient, `unnecessary rerenders can lead to jank, layout shifts, and wasted computation`.

### ⚙️ The Three Phases of React's Render Cycle

React’s internal render cycle can be broken down into three key phases (with a passive phase in between):

#### 1. Render Phase

- React calls the render functions of components (functional or class-based).
- It creates a “work-in-progress” tree and compares the result with the previous render output (diffing).
- This phase is pure: it `should not include side effects`.

#### 2. Commit Phase

- React takes the list of changes and applies them to the DOM.
- Refs are also updated in this phase.
- This is a mutating phase where side effects can occur.

#### (Between Commit & Cleanup) Passive Effects Phase

- useEffect callbacks run here.
- If a useEffect triggers a setState, the render cycle starts again.

#### 3. Cleanup Phase

- React performs cleanup inside the next useEffect call, not as a separate third phase

### 🌀 Where Do Animations Fit?

- CSS animations → Often run on a separate thread (GPU-accelerated).
- JavaScript animations (manual DOM mutations) → Happen during the commit phase or in rerenders.
- Changing state on an interval (like setInterval) triggers rerenders repeatedly, which can overload the render-commit-effect cycle if not throttled.

### 🧠 Component Rerendering Logic

- A state change or prop change will rerender the component and all its descendants.
- React doesn’t guess which subtrees changed—it assumes all children must re-render unless told otherwise.

#### Example:

```plaintext
Component Tree:
App
├── A (state changes here)
│   ├── B
│   └── C
```

If A changes, B and C rerender, unless memoized.

### 📦 React 17 vs 18: Batching

| Trigger Type           | React 17 Behavior | React 18 Behavior       |
| ---------------------- | ----------------- | ----------------------- |
| Synthetic events       | ✅ batched        | ✅ batched              |
| `setTimeout` / `fetch` | ❌ not batched    | ✅ batched              |
| Native DOM events      | ❌ not batched    | ✅ batched (if wrapped) |

### 📉 How to Improve Render Performance

1. Push state down in the component tree.

   - Avoid lifting state higher than necessary.
   - From the point of change, React rerenders everything downward.

2. Prevent unnecessary rerenders

   - Use React.memo() (functional components).
   - Use shouldComponentUpdate or extend PureComponent (class components).
   - Use useMemo, useCallback to memoize expensive calculations or functions.
   - Use useRef to persist values across renders without triggering one.

### 🔍 Why "unchanged props = no rerender" is not always true

- The assumption only holds if:

  - You use React.memo or similar.
  - The component performs a shallow comparison and skips updates when props didn’t change.

> Without explicit memoization or shallow compare, components rerender if their parent does, even if props haven’t changed.

### 🔁 Summary Flow

![](https://i.imgur.com/cn43gQE.jpeg)

---

## 1.4 - React Fiber

### 🔧 React Fiber and Virtual DOM

- Fiber is the internal data structure React uses to represent components. It tracks relationships like siblings and children and allows React to pause, resume, and optimize rendering.
- `React is no longer "just a virtual DOM"`. It has grown significantly since its early days, and optimizations like Fiber make it more sophisticated.

![](https://i.imgur.com/iRFnIwn.png)

### 🔑 Importance of Keys in Lists

- React uses keys to track elements across renders. `Good keys help React optimize DOM updates instead of re-rendering everything`.
- `Avoid using Math.random() or array indexes as keys`:

  - Math.random() is inconsistent.
  - Indexes change with sorting or filtering, breaking tracking.

> Array index as key is safe only if the list never changes in order, length, or content. Otherwise, use stable unique IDs.

- Best practice: assign `a consistent and unique ID to each item` when the data is fetched or constructed. If the data lacks IDs, map over it to assign stable ones (e.g., `UUIDs or counters`).
- Using duplicate keys (e.g., based on template names) can silently break rendering and lead to hard-to-diagnose bugs.

### 🧠 React Rendering Heuristics

- Avoid excessive or unnecessary DOM changes. For instance, changing an element from a div to a span dynamically can confuse React and hurt performance.
- `Use consistent semantic HTML`, but don’t abuse clever tricks just to be "smart"; React relies on predictable patterns.
- Sometimes React will re-render a full section, sometimes just part of it. It depends on the heuristics and what signals (like keys) you give it.
- `React uses a diffing algorithm (reconciliation) based on element types and keys`. Predictable, consistent structures help avoid unnecessary DOM mutations.

### ⚠️ Context API Gotchas

- Lifting state via Context can cause unintentional re-renders across your app.
- Wrapping large parts of the component tree with context providers will make changes cascade unless you're memoizing effectively or using patterns like splitting providers.
- `useContext triggers a rerender whenever the context value changes`, even if only part of it is used. To avoid this `use value memoization or split context into smaller ones`.
- Libraries like Redux (especially with `useSyncExternalStore` in React 18) can avoid these problems because they're optimized differently.
