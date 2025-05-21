# 4 - Suspense & Transitions

## 4.1 - Suspense (Component-Level Code Splitting)

Suspense in React is a `set of APIs designed to handle asynchronous behavior`, particularly around loading components or data. While some parts are still experimental or framework-specific (like data fetching), lazy-loading components with Suspense is fully supported today.

### Why is Suspense useful?

- You don’t need to load all components for every page upfront.
- You can `progressively load parts of your application as users need them`, improving initial load performance.
- Useful especially for rarely used parts of the app, like settings or admin pages.

### Key Concepts

- Suspense Boundary: Like an error boundary, but used to wrap lazy-loaded components. Shows a fallback UI (like a loading spinner) while the component is being fetched.
- React.lazy(): Allows you to dynamically import components. Works well with Suspense to delay loading until needed.
- import(): The native ES module function used behind the scenes to dynamically fetch a file.

Here’s how you might use it in a notes app:

```tsx
import { lazy, Suspense } from "react";

const Editor = lazy(() => simulateNetwork(() => import("./Editor")));

function NoteView({ isEditing }) {
  if (isEditing) {
    return (
      <Suspense fallback={<LoadingIndicator />}>
        <Editor />
      </Suspense>
    );
  }

  return <NoteContent />;
}
```

> simulateNetwork is just a mock to delay loading for demo purposes.

### Performance Benefit

JavaScript bundles take longer to load and parse than assets like images. Loading everything upfront hurts performance. Suspense lets you:

- Keep the initial bundle smaller.
- Improve Time-to-Interactive.
- Dynamically load parts only when needed (e.g., only load the editor when "Edit" is clicked).

### Real-world usage

Even in production apps, you could:

- `Delay loading parts of the UI that are not critical to the initial view`.
- Preload code on hover or interaction intent to avoid jank.
- `Combine with routing systems like React Router or Next.js for page-level code splitting`.

### What's Not Ready Yet?

- Suspense for data fetching is still not available in plain React apps.

  - Some frameworks like Next.js or Remix support it.
  - It’s in active development and will allow you to suspend while waiting for async data (instead of manually setting loading/error states).
