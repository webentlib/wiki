Svelte: Best practice to combine $derived and $state
====================================================

**First approximation:**
```js
let { data } = $props();
let posts = $derived.by(() => {
    let posts = $state(data.posts);
    return posts;
});
```
It is not documented, and me personally obtained it here:
https://discord.com/channels/457912077277855764/1406193849423888415/1406241130726690837
From _mjadobson_, approved by _Patrick_.

**Others advice, for semantics and linters, that `let` better be `const`:**
```js
let posts = $derived.by(() => {
    const posts = $state(data.posts);
    return posts;
});
```
Performance may depend on an engine.

**Another way to make it one line:**
```js
let posts = $derived.by(() => { const _ = $state(data.posts); return _ });
```
And now, while copy-paste, there are 2 places left to change, not 4.

**For sure, it COULD NOT be:**
```js
let posts = $derived.by(() => $state(data.posts));
```
Since:
> `$state(...)` can only be used as a variable declaration initializer, a class field declaration, or the first assignment to a class field at the top level of the constructor.

---

_Probably it'll be nice to have `$reactive` rune one day, combining both or something_

---

I've one have more elegant solution — please share
Thx