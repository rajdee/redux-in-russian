# Действия (Actions)

Во-первых, давайте определим некоторые действия.

**Действия** - это структура, которая передает данные из вашего приложения в хранилище. Они являются *единственным* источником информации для хранилища. Вы отправляете их в хранилище используя метод [`store.dispatch()`](../api/Store.md#dispatch).

Например, вот действие которое представляет добавление нового пункта в список дел:

```js
{
  type: 'ADD_TODO',
  text: 'Build my first Redux app'
}
```

Действия - это обычный объект JavaScript. По соглашению, действия должны иметь строковое поле `type`, которое указывает на тип исполняемого действия. Типы должны, как правило, определяться как строковые константы. После того, как ваше приложение разрастется, вы можете захотеть переместить их в отдельный модуль.

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes';
```

>##### Примечание к шаблону разработки

>Вам не нужно определять константы типа действий в отдельном файле или даже определить их и вовсе. Для небольшого проекта, было бы проще просто использовать строковые литералы для типов действий. Однако, есть некоторые преимущества в явном объявлении констант в больших проектах. Прочтите [Reducing Boilerplate](../recipes/ReducingBoilerplate.md) для знакомства с большим количеством практических советов, позволяющих хранить вашу кодовую базу в чистоте.

Кроме `type`, структуру объекта действий вы можете строить на ваше усмотрение. Если вам интересно, изучите [Flux Standard Action](https://github.com/acdlite/flux-standard-action) для рекомендаций о том, как могут строится действия.

Мы добавим еще один тип действия для описания пометки задачи, как выполненной. Мы обращаемся к конкретному todo по `index`, потому что мы храним их в виде массива. В реальном приложении, разумнее генерировать уникальный ID каждый раз, когда что-то новое создается.

```js
{
  type: COMPLETE_TODO,
  index: 5
}
```

Это хорошая идея, передавать как можно меньше данных в каждом действии. Например, лучше отправить `index`, чем весь объект todo.

Наконец, мы добавим еще один тип действия для редактирования видимых в данный момент задач.

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Генераторы действий (Action Creators)

**Action creators** - не что иное, как функции, которые создают действия. Довольно просто запутаться в терминах “action” and “action creator,” поэтому постарайтесь использовать правильный термин.

В [традиционной реализации Flux](http://facebook.github.io/flux), action creators при выполнении часто вызывают dispatch, примерно так:

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  };
  dispatch(action);
}
```

В противоположность этому, в Redux action creators являются  **чистыми функциями** с нулевыми сайд-эффектами. Они просто возвращают action:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  };
}
```

Это делает их более подвижными (portable ориг.) и легкими для тестирования. На самом деле, чтобы запустить отправку - передайте результат action creator в функцию `dispatch()`:

```js
dispatch(addTodo(text));
dispatch(completeTodo(index));
```

Или создайте **связанный генератор действия (bound action creator)** который автоматически запускает отправку действия:

```js
const boundAddTodo = (text) => dispatch(addTodo(text));
const boundCompleteTodo = (index) => dispatch(completeTodo(index));
```

Вы сразу же сможете его вызвать:

```
boundAddTodo(text);
boundCompleteTodo(index);
```

Доступ к функции `dispatch()` может быть получен непосредственно из хранилища (store) [`store.dispatch()`](../api/Store.md#dispatch), но, что более вероятно, вы будете получать доступ к ней при помощи чего-то типа `connect()` из [react-redux](http://github.com/gaearon/react-redux). Вы можете использовать функцию [`bindActionCreators()`](../api/bindActionCreators.md) для автоматического привязывания большого количества генераторов действий (action creators) к функции `dispatch()`.

## Исходный код

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

Теперь давайте [создадим несколько редьюсеров](Reducers.md) для того, чтобы описать как будет обновляться состояние (state) когда мы отправляем действия (actions)!

>##### Для продвинутых пользователей
>Если Вы уже знакомы с базовыми концептами и уже освоили этот обучающий материал, то не забудьте заценить [асинхронные действия (async actions)](../advanced/AsyncActions.md) в [руководстве для опытных](../advanced/README.md), чтобы научиться работать с AJAX ответами и создавать генераторы действий для асинхронного потока управления.
