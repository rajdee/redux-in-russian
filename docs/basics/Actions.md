# Действия

Во-первых, давайте определим некоторые действия.

**Действия** - это структура, которая передает данные из вашего приложения в хранилище. Они являются *единственным* источником информации для хранилища. Вы отправляете их в хранилище используя метод [`store.dispatch()`](../api/Store.md#dispatch).

Вот пример, которое представляет действие, добавляющее новый пункт в список дел:

```js
{
  type: 'ADD_TODO',
  text: 'Build my first Redux app'
}
```

Действия - это обычный объект JavaScript. По соглашению, действия должны иметь строковое поле `type`  которое указывает на тип исполняемого действия. Типы должны, как правило, определятся как строковые константы. После того, как ваше приложение разрастется, вы можете переместить их в отдельный модуль.

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes';
```

>##### Примечание к шаблону разработки

>Вам не нужно определять константы типа действий в отдельном файле или даже определить их и вовсе. Для небольшого проекта, было бы проще просто использовать строковые литералы для типов действий. Однако, есть некоторые преимущества в явном объявлении констант в больших проектах. Прочтите [Reducing Boilerplate](../recipes/ReducingBoilerplate.md) для знакомства с большим количеством практических советов, позволяющих хранить вашу кодовую базу в чистоте.

Кроме `type`, структуру объекта действий вы можете строить на ваше усмотрение. Если вам интересно, изучите [Flux Standard Action](https://github.com/acdlite/flux-standard-action) для рекомендаций, как должны строятся действия.

Мы добавим еще один тип действия для описывания отметки задачи, как выполненной. Мы обращаемся к конкретному todo по `index`, потому что мы храним их в виде массива.  В реальном приложении, разумнее генерировать уникальный ID каждый раз, когда что-то новое создается.

```js
{
  type: COMPLETE_TODO,
  index: 5
}
```

Это хорошая идея, передавать как можно меньше данных в каждом действии. Например, лучше отправить `index`, чем весь объект todo.

Наконец, мы добавим еще один тип действия по изменению видимых, в данный момент, задач.

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Action Creators

**Action creators** are exactly that—functions that create actions. It's easy to conflate the terms “action” and “action creator,” so do your best to use the proper term.

In [traditional Flux](http://facebook.github.io/flux) implementations, action creators often trigger a dispatch when invoked, like so:

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  };
  dispatch(action);
}
```

By contrast, in Redux action creators are **pure functions** with zero side-effects. They simply return an action:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  };
}
```

This makes them more portable and easier to test. To actually initiate a dispatch, pass the result to the `dispatch()` function:

```js
dispatch(addTodo(text));
dispatch(completeTodo(index));
```

Or create a **bound action creator** that automatically dispatches:

```js
const boundAddTodo = (text) => dispatch(addTodo(text));
const boundCompleteTodo = (index) => dispatch(completeTodo(index));
```

You’ll be able to call them directly:

```
boundAddTodo(text);
boundCompleteTodo(index);
```

The `dispatch()` function can be accessed directly from the store as [`store.dispatch()`](../api/Store.md#dispatch), but more likely you'll access it using a helper like [react-redux](http://github.com/gaearon/react-redux)'s `connect()`. You can use [`bindActionCreators()`](../api/bindActionCreators.md) to automatically bind many action creators to a `dispatch()` function.

## Source Code

### `actions.js`

```js
/*
 * action types
 */

export const ADD_TODO = 'ADD_TODO';
export const COMPLETE_TODO = 'COMPLETE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';

/*
 * other constants
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
};

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text };
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index };
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter };
}
```

## Next Steps

Now let’s [define some reducers](Reducers.md) to specify how the state updates when you dispatch these actions!

>##### Note for Advanced Users
>If you’re already familiar with the basic concepts and have previously completed this tutorial, don’t forget to check out [async actions](../advanced/AsyncActions.md) in the [advanced tutorial](../advanced/README.md) to learn how to handle AJAX responses and compose action creators into async control flow.
