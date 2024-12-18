---
title: Conventions
description: Conventions in effector from core team for stores, events and effects
redirectFrom:
  - /en/docs/conventions/naming
  - /en/conventions/naming
---

# Conventions in effector (#effector-conventions)

First of all, to avoid any misconceptions and get better developer experience for all of us. This document contains several pretty simple rules to keep consistency between different projects written with effector.

## Naming convention (#naming-convention)

### Stores naming (#stores-naming)

Your stores should be distinguished by a `$`. The choice between prefix or postfix is mostly a matter of personal preference. This should be done to have a better search experience in your IDE.

```js
const $user = createStore({});
```

### Effect naming (#effects-naming)

It is recommended to add the `Fx` postfix to the end of your effects. It will let you differentiate your effects from the events.

```js
const fetchUserFx = createEffect(async () => {
  const res = await fetch("my pretty url");
  return res.json();
});
```

### Event naming (#events-naming)

There are no any specific rules for this. However, the proposal is to name events that directly trigger store updates, as like as they have already happened.

```js
const emailChanged = createEvent();

$user.on(emailChanged, (user, email) => ({
  ...user,
  email,
}));
```
