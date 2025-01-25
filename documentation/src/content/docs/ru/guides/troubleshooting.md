---
title: Исправление ошибок в Effector
description: Исправление ошибок
---

# Исправление ошибок (#troubleshooting)

## Основные ошибки (#common-errors)

### `store: undefined is used to skip updates` (#store-undefined)

Ошибка вида: `store: undefined is used to skip updates. To allow undefined as a value provide explicit { skipVoid: false } option` сообщает вам о том, что вы пытаетесь передать в ваш стор значение `undefined`, что, возможно, является некорректным поведением.

Если вам действительно нужно передать ваш стор значение `undefined`, то вам надо вторым аргументом в `createStore` передать объект со свойством `skipVoid: false`.

```ts
const $store = createStore(0, {
  skipVoid: false,
});
```

### `There is a store without sid in this scope, its value is omitted` (#store-without-sid)

Эта ошибка часто встречается при работе с SSR, она связана с тем, что у вашего стора отсутствует `sid` (stable id), который необходим для корректной гидрации данных с сервера на клиент.
Чтобы исправить эту проблему вам нужно добавить этот `sid`.<br/>
Сделать это вы можете несколькими способами:

1. Использовать [babel](/ru/api/effector/babel-plugin/) или [SWC](/ru/api/effector/swc-plugin/) плагин
2. Добавить `sid` в ручную, передав во второй аргумент `createStore` объект со свойством `sid`:
   ```ts
   const $store = createStore(0, {
     sid: "unique id",
   });
   ```

Более подробно [про `sid`, как это работает и зачем это нужно](/ru/explanation/sids).

### `scopeBind: scope not found` (#scope-not-found)

Эта ошибка случается когда скоуп потерялся на каком-то из этапов выполнения и `scopeBind` не может связать событие или эффект с нужным скоупом выполнения.<br/>
Эта ошибка могла быть вызвана:

1. Вы используете режим работы 'без скоупа' и у вас их нет в приложении
2. Ваши [юниты были вызваны вне скоупа](#using-using-without-use-unit)

Возможные решения:

1. Используйте `scopeBind` внутри эффектов:

   ```ts
   const event = createEvent();

   // ❌ - не вызывайте scopeBind внутри колбеков
   const effectFx = createEffect(() => {
     setTimeout(() => {
       scopeBind(event)();
     }, 1111);
   });

   // ✅ - используйте scopeBind внутри эффекта
   const effectFx = createEffect(() => {
     const scopeEvent = scopeBind(event);

     setTimeout(() => {
       scopeEvent();
     }, 1111);
   });
   ```

2. Ваши юниты должны быть вызваны внутри скоупа:
   - При работе с фреймворком используйте `useUnit`
   - Если у вас происходит вызов события или эффекта вне фреймворка, то используйте `allSettled` и передайте нужный `scope` в аргумент

Если того требует ваша реализация, а от ошибки нужно избавиться, то вы можете передать свойство `safe:true` во второй аргумент метода.

```ts
const scopeEvent = scopeBind(event, {
  safe: true,
});
```

## Частые проблемы (#gotchas)

### Мое состояние не изменилось (#my-state-not-changed)

Скорее всего вы работаете со скоупами и в какой-то момент ваш скоуп для юнита потерялся из-за чего он запустился в глобальной области.<br/>
Это происходит при передаче юнитов (события или эффекты) в колбэк внешних функций, таких как:

- `setTimeout`/`setInterval`
- `addEventListener`
- `webSocket`

Чтобы исправить эту проблему привяжите ваше событие или эффект к текущему скоупу при помощи [`scopeBind`](/ru/api/effector/scopeBind):

```ts
const event = createEvent();

// ❌ - так у вас событие вызовется в глобальной области видимости
setTimeout(() => {
  event();
}, 1000);

// ✅ - так у вас будет работать как ожидаемо
const scopeEvent = scopeBind(event);
setTimeout(() => {
  scopeEvent();
}, 1000);
```

#### Использование юнитов без `useUnit` (#using-units-without-use-unit)

Возможно вы используете события или эффекты во фреймворках без использования хука `useUnit`, что может также повлиять на неправильную работу со скоупами.<br/>
Чтобы исправить это поведение передайте нужный юнит в `useUnit` хук и используйте возвращаемое значение.

:::info{title="Информация"}
[Использования хука `useUnit` является рекомендованным способом работы](/ru/guides/best-practices#use-unit) с юнитами.
:::

[Что такое потеря скоупа и почему это происходит](/ru/api/effector/Scope#scope-loss)

## Не нашли ответ на свой вопрос ? (#no-answers)

Если вы не нашли ответ на свой вопрос, то вы всегда можете задать сообществу:

- RU Telegram - https://t.me/effector_ru
- EN Telegram - https://t.me/effector_en
- Discord - https://discord.gg/t3KkcQdt
- Reddit — [reddit.com/r/effectorjs](https://www.reddit.com/r/effectorjs/)