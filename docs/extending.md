# Extending FingerprintJS

You can exclude built-in entropy components and add custom entropy components,
for example, when a component is too unstable for your needs.
Use the built-in hash function to hash your custom list of components.

## Excluding built-in components at collection time

Pass the `excludeSources` option to `load()` to skip collecting the given built-in entropy sources.
Excluded sources are never loaded, so the collection takes less time and CPU — this matters on low-end devices
and for the expensive sources such as `fonts`.

```js
// The `fonts` and `audio` components will not be collected at all
const fpPromise = FingerprintJS.load({ excludeSources: ['fonts', 'audio'] })
```

The visitor identifier and the returned `components` are computed from the remaining components only,
so excluding a source changes the visitor identifiers. The confidence score keeps working as usual,
even when the `platform` source (which the score reads) is excluded.

An example of component exclusion after collection:

```js
// ...

const result = await fp.get()

// The `languages` and `audio` components will be excluded
const { languages, audio, ...components } = result.components

// Optionally, you can make a visitor identifier from your custom list of components
const visitorId = FingerprintJS.hashComponents(components)
```

An example of component addition:

```js
// ...

const result = await fp.get()

// New components will be added: `foo` and `bar`.
// You should implement the `getFooComponent` and `getBarComponent` functions by yourself,
// they can return any value.
const components = {
  ...result.components,
  foo: { value: await getFooComponent() },
  bar: { value: await getBarComponent() },
}

// Optionally, you can make a visitor identifier from your custom list of components
const visitorId = FingerprintJS.hashComponents(components)
```

An example of both component exclusion and addition:

```js
// ...

const result = await fp.get()

// The `languages` and `audio` components will be excluded
const { languages, audio, ...components } = result.components

// New components will be added: `foo` and `bar`
const extendedComponents = {
  ...components,
  foo: { value: await getFooComponent() },
  bar: { value: await getBarComponent() },
}

// Optionally, you can make a visitor identifier from your custom list of components
const visitorId = FingerprintJS.hashComponents(extendedComponents)
```

You can format your custom list of components into a human-friendly text:

```js
// ...

const debugOutput = document.querySelector('pre')
debugOutput.textContent = FingerprintJS.componentsToDebugString(components)
```

## Canvas stabilization

The [canvas entropy source](https://github.com/fingerprintjs/fingerprintjs/blob/master/src/sources/canvas.ts) consists of 2 images.
The first (with a geometry only) is more stable, the second (with a text) gives more entropy.
Agent uses both the images by default.
If you need more stability, you can exclude the text image.
Example:

```js
// ...

let { components } = await fp.get()

if ('value' in components.canvas) {
  components.canvas.value.text = ''
}

// Optionally, you can make a visitor identifier from your custom list of components
const visitorId = FingerprintJS.hashComponents(components)
```
