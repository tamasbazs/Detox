# Expect

Detox uses [matchers](APIRef.Matchers.md) to match UI elements in your app and expectations to verify those elements are in the expected state.

Use [actions]('APIRef.ActionsOnElement.md') to simulate use interaction with elements.

## Methods

- [`.toBeVisible()`](#tobevisible)
- [`.toExist()`](#toexist)
- [`.toHaveText()`](#tohavetexttext)
- [`.toHaveLabel()`](#tohavelabellabel)
- [`.toHaveId()`](#tohaveidid)
- [`.toHaveValue()`](#tohavevaluevalue)
- [`.toHaveSliderPosition()`](#tohavesliderpositionnormalizedposition-tolerance--ios-only) **iOS only**
- [`.not`](#not)
- [`.withTimeout()`](#withtimeouttimeout)

### `toBeVisible()`

Expects the element to be visible on screen.

On iOS, visibility is defined by having the view, or one of its subviews, be topmost at the view's activation point on screen.

```js
await expect(element(by.id('UniqueId204'))).toBeVisible();
```

### `toExist()`
Expects the element to exist within the app’s current UI hierarchy.

```js
await expect(element(by.id('UniqueId205'))).toExist();
```

### `toHaveText(text)`
Expects the element to have the specified text.

```js
await expect(element(by.id('UniqueId204'))).toHaveText('I contain some text');
```

### `toHaveLabel(label)`

Expects the element to have the specified label as its accessibility label (iOS) or content description (Android). In React Native, this corresponds to the value in the [`accessibilityLabel`](https://facebook.github.io/react-native/docs/view.html#accessibilitylabel) prop.

```js
await expect(element(by.id('UniqueId204'))).toHaveLabel('Done');
```

### `toHaveId(id)`

Expects the element to have the specified accessibility identifier. In React Native, this corresponds to the value in the [`testID`](https://reactnative.dev/docs/view.html#testid) prop.

```js
await expect(element(by.text('I contain some text'))).toHaveId('UniqueId204');
```

### `toHaveValue(value)`

Expects the element to have the specified accessibility value. In React Native, this corresponds to the value in the [`accessibilityValue`](https://reactnative.dev/docs/view.html#accessibilityvalue) prop.

```js
await expect(element(by.id('UniqueId533'))).toHaveValue('0');
```

### `toHaveSliderPosition(normalizedPosition, tolerance)`  iOS only

Expects the slider element to have the specified normalized position ([0, 1]), within the provided tolerance (optional).

```js
await expect(element(by.id('slider'))).toHaveSliderPosition(0.75);
await expect(element(by.id('slider'))).toHaveSliderPosition(0.3113, 0.00001);
```

### `not`

Negates the expectation.

```js
await expect(element(by.id('UniqueId533'))).not.toBeVisible();
```

### `withTimeout(timeout)`

Waits until the expectation is resolved for the specified amount of time. If timeout is reached before resolution, the expectation is failed.

`timeout`—the timeout to wait, in ms

```js
await waitFor(element(by.id('UniqueId204'))).toBeVisible().withTimeout(2000);
```

## Deprecated Methods

- [`.toBeNotVisible()`](#tobenotvisible)
- [`.toNotExist()`](#tonotexist)

### `toBeNotVisible()`

**Deprecated:** Use `.not.toBeVisible()` instead.

Expects the element to not be visible on screen.

```js
await expect(element(by.id('UniqueId205'))).toBeNotVisible();
```

### `toNotExist()`

**Deprecated:** Use `.not.toExist()` instead.

Expects the element to not exist within the app’s current UI hierarchy.

```js
await expect(element(by.id('RandomJunk959'))).toNotExist();
```
