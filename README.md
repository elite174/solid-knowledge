# Hidden knowlege about solid-js

This repo contains non-obvious things, facts, gotchas and other useful knowledge related to `solid-js` (library itself, router, solid-start).

Feel free to contribute.

## Solid-js

### `Suspense` and accessing resources inside primitives

Accessing resource inside `createEffect` **won't trigger** `Suspense`, but accessing it inside `createComputed` or `createMemo` **will trigger** `Suspense`.

Example:

```tsx
import { render } from "solid-js/web";
import { createSignal, createResource, Suspense, createComputed, createEffect } from "solid-js";

function Example() {
  const [count, setCount] = createSignal(2);

  const [data] = createResource(
    () =>
      new Promise((resolve) => {
        setTimeout(() => resolve(1), 3000);
      })
  );

  // `createEffect` won't trigger `Suspense`,
  // but `createComputed` or `createMemo` will trigger.
  createEffect(() => {
    const value = data();

    if (value) setCount(value);
  });

  return (
    <button type="button" onClick={() => setCount((c) => c + 1)}>
      {count()}
    </button>
  );
}

render(
  () => (
    <Suspense fallback="loading">
      <Example />
    </Suspense>
  ),
  document.getElementById("app")!
);
```

Possible explanation is [here](https://discord.com/channels/722131463138705510/722131463889223772/1261153998950371419)
