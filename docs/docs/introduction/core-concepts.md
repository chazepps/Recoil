---
title: Core Concepts
---

## Overview

Recoil lets you create a data-flow graph that flows from *atoms* (shared state) through *selectors* (pure functions) and down into your React components.
Atoms are units of state that components can subscribe to. Selectors transform

## Atoms

Atoms are units of state. They're updateable and subscribeable: when an atom is updated, each subscribed component is re-rendered with the new value.

Atoms can be used in place of React local component state. If the same atom is used from multiple components, all those components share their state.

Atoms are created using the `atom` function:

```javascript
const fontSizeState = atom({
  key: 'fontSizeState',
  default: 14,
});
```

Atoms need a unique key, which is used for debugging, persistence, and for certain advanced APIs that let you see a map of all atoms. It is a fatal error
for two atoms to have the same key, so make sure they're globally unique. Like React component state, they also have a default value.

To read and write an atom from a component, we use a hook called `useRecoilState`. It's just like React's `useState`, but now the state can be shared between components:

```jsx
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return (
    <button onClick={() => setFontSize(size => size + 1)} style={{fontSize}}>
      Click to Enlarge
    </button>
  );
}
```

Clicking on the button will increase the font size of the button by one. But now some other component can also use the same font size:

```jsx
function Text() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return (
    <p style={{fontSize}}>
      This text will increase in size too.
    </p>
  );
}
```


## Selectors

A **selector** is a pure function that accepts atoms or other selectors as input. When upstream atoms or selectors are updated, the selector
function will be re-evaluated. Components can subscribe to selectors just like atoms, and will then be re-rendered when those selectors change.

Selectors are used to calculate derived data that is based on state. By avoiding redundant state, this usually obviates the need for
reducers to keep state in sync and valid. Instead, a minimal set of state is stored in atoms, while everything else is efficiently computed
as a function of that minimal state. Since selectors keep track of what components need them and what state they depend on, they make this
functional approach efficient.

From the point of view of components, selectors and atoms have the same interface and can therefore be substituted for one another.

Selectors are defined using the `selector` function:

```javascript
const fontSizeLabelState = selector({
  key: 'fontSizeLabelState',
  get: ({get}) => {
    const fontSize = get(fontSizeState);
    const unit = 'px';

    return `${fontSize}${unit}`;
  },
});
```

Like atoms, selectors must have a unique key. The `get` property is the function that is to be computed. It can access the value of atoms and other
selectors using the `get` argument passed to it. Whenever it accesses another atom or selector, a dependency relationship is created such that
updating that atom or selector will cause the function to be recomputed.

In this `fontSizeLabelState` example, the selector has one dependency: the `fontSizeState` atom. Conceptually, the `fontSizeLabelState` selector behaves like a pure function that takes a `fontSizeState` as input and returns a formatted font size label as output.

Selectors

Selectors can be read using `useRecoilValue()`, which takes an atom/selector as its first parameter and returns the corresponding value. Note we can't use `useRecoilState()` as the `fontSizeLabelState` selector is not writeable (see the [selector API reference](/docs/api-reference/core/selector) for more information on writeable selectors):

```jsx
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  const increaseFontSizeByOne = () => setFontSize(fontSize + 1);

  const fontSizeLabel = useRecoilValue(fontSizeLabelState);

  return (
    <>
      <div>Current font size: ${fontSizeLabel}</div>

      <button onClick={increaseFontSizeByOne} style={{fontSize}}>
        Click Me!
      </button>
    </>
  );
}
```

Clicking on the button now does two things: it increases the font size of the button while also updating the font size label to reflect the current font size.