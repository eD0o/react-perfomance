# 🎓 React Performance Course – Introduction by Steve (Head of Frontend at Temporal)

## 🧠 What This Course Is About

This course `isn't about sprinkling React.memo and hoping for the best`. It's about understanding React’s internals and learning to solve real-world performance problems with purpose.

You’ll build the skills to:

- Structure components and state effectively, so you often avoid performance issues entirely.
- Diagnose what’s actually slowing your app down — and `know when optimizations like memoization are worth it`.
- Use React 18 `features like startTransition, Suspense, useDeferredValue, and useTransition` in meaningful ways.

### 🗺️ Key Principles

1. Shape first, optimize second
   If you can solve a problem by reorganizing your component hierarchy or managing state better—do that first.

2. Don’t overuse memoization
   `Memoization is great—but only when the cost of checking is lower than the cost of re-rendering`. Don’t blindly wrap everything in React.memo.

3. Progressive loading with Suspense
   The Suspense API is a solid way to progressively load parts of your app. It’s a good idea — and it’s getting even better.

4. Reach for transitions when needed
   The `Transition API is there to help when you're in a tight spot`. It shines in situations where you need to keep your UI responsive under load.
