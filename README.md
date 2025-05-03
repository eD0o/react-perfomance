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

### 🧠 Final Thoughts

- Performance is about doing less, not thinking harder
- `Preventing unnecessary work is better than optimizing necessary work`
- Only optimize when there's a measurable problem
- Don’t trust blogs blindly use actual performance data
