# Экшены (Actions)

Во-первых, давайте определим некоторые экшены.

**Экшены** — это структуры, которые передают данные из вашего приложения в стор. Они являются *единственными* источниками информации для стора. Вы отправляете их в стор, используя метод [`store.dispatch()`](../api/Store.md#dispatch).

Например, вот экшен, которое представляет добавление нового пункта в список дел:

```js
const ADD_TODO = 'ADD_TODO'
```

```js
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

Экшены — это обычные JavaScript-объекты. Экшены должны иметь поле `type`, которое указывает на тип исполняемого экшена. Типы должны быть, как правило, заданы, как строковые константы. После того, как ваше приложение разрастется, вы можете захотеть переместить их в отдельный модуль.

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

>##### Примечание к шаблону разработки

>Вам не нужно определять константы типа экшенов в отдельном файле или даже определять их и вовсе. Для небольшого проекта было бы проще просто использовать строковые литералы для типов экшенов. Однако есть некоторые преимущества в явном объявлении констант в больших проектах. Прочтите [Reducing Boilerplate](../recipes/ReducingBoilerplate.md) для знакомства с большим количеством практических советов, позволяющих хранить вашу кодовую базу в чистоте.

Кроме `type`, структуру объекта экшенов вы можете строить на ваше усмотрение. Если вам интересно, изучите [Flux Standard Action](https://github.com/acdlite/flux-standard-action) для рекомендаций о том, как могут строится экшены.

Мы добавим еще один тип экшена, который будет отмечать задачу, как выполненную. Мы обращаемся к конкретному todo по `index`, потому что мы храним их в виде массива. В реальном приложении разумнее генерировать уникальный ID каждый раз, когда что-то новое создается.

```js
{
  type: TOGGLE_TODO,
  index: 5
}
```

Это хорошая идея, передавать как можно меньше данных в каждом экшене. Например, лучше отправить `index`, чем весь объект todo.

Наконец, мы добавим еще один тип экшена для изменения видимых, в данный момент, задач.

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Генераторы экшенов (Action Creators)

**Генераторы экшенов (Action Creators)** — не что иное, как функции, которые создают экшены. Довольно просто путать термины “action” и “action creator,” поэтому постарайтесь использовать правильный термин.

В Redux генераторы экшенов (action creators) просто возвращают action:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

Это делает их более переносимыми и легкими для тестирования.

В [традиционной реализации Flux](http://facebook.github.io/flux), генераторы экшенов (action creators) при выполнении часто вызывают dispatch, примерно так:

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

В Redux это *не* так.
Вместо того чтобы на самом деле начать отправку, передайте результат в функцию `dispatch()`:

```js
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

Кроме того, вы можете создать **связанный генератор экшена (bound action creator)**, который автоматически запускает отправку экшена:

```js
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))
```

Теперь вы можете вызвать его напрямую:

```
boundAddTodo(text)
boundCompleteTodo(index)
```

Доступ к функции `dispatch()` может быть получен непосредственно из стора (store) [`store.dispatch()`](../api/Store.md#dispatch), но, что более вероятно, вы будете получать доступ к ней при помощи чего-то типа `connect()` из [react-redux](http://github.com/gaearon/react-redux). Вы можете использовать функцию [`bindActionCreators()`](../api/bindActionCreators.md) для автоматического привязывания большого количества генераторов экшенов (action creators) к функции `dispatch()`.

Генератор экшены так же могут быть асинхронными и иметь сайд-эффекты. Вы можете почитать про [асинхронные экшены](../advanced/AsyncActions.md) в [расширенном руководстве](../advanced/README.md), чтобы узнать, как обрабатывать ответы AJAX и создавать генераторы действий в асинхронном потоке управления. Не переходите к асинхронным экшенам до тех пор, пока вы не завершите базовое руководство, так как оно охватывает другие важные концепции, которые необходимы для продвинутого руководства и асинхронных экшенов.

## Исходный код

### `actions.js`

```js
/*
 * типы экшенов
 */

export const ADD_TODO = 'ADD_TODO'
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * другие константы
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * генераторы экшенов
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

## Дальнейшие шаги

Теперь давайте [создадим несколько редьюсеров](Reducers.md) для того, чтобы описать как будет обновляться состояние (state), когда мы отправляем эти экшены (actions)!
