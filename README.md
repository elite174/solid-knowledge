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

## Solid-js router

### What is `createAsync`?

`createAsync` is just a wrapper (for now, in Solid 2.0 there'll be a change for that) over `createResource` ([source code](https://github.com/solidjs/solid-router/blob/main/src/data/createAsync.ts)). So when you call accessor from `createAsync` you **will trigger `Suspense`**:

```tsx
const data = createAsync(() => fetch(...));

return <Suspense fallback="loading...">
   {/** This will trigger Suspense all the time when createAsync updates */
   {data()}
</Suspense>
```

### How to avoid triggering `Suspense` with `createAsync`?

UPDATE for router `^0.14.2`: `createAsync` preserves `.latest` field (as for resources), so feel free to use this.

For `createResource` we have `.latest` property which doesn't trigger `Suspense` on refetching. However there's no such thing for `createAsync`, what should we do in this case?

There's a small [hack](https://discord.com/channels/722131463138705510/1260246424508170321):

```tsx
function latest<T>(signal: Accessor<T | undefined>) {
  const [latest, setLatest] = createSignal(createRoot(signal));

  createRoot(() => {
    createMemo(() => setLatest(signal));
  });

  if (latest() === undefined) return signal();
  return latest();
}
```

The point is that we're moving suspense tracking to another root, thus avoiding triggering suspense for the current root.

And you can use it inside JSX:

```tsx
const data = createAsync(() => fetch(...));

return <Suspense fallback="loading...">
   {/** This won't trigger Suspense when createAsync updates */
   {latest(data)}
</Suspense>
```
