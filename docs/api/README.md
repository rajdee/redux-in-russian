# API

Redux имеет скромный API . Он определяет лишь набор соглашений по использованию, например, [reducers](../Glossary.md#reducer) и предлагает несколько вспомогательных функций для того, чтобы объединить эти соглашения.

В этом разделе полностью будет описан Redux API. Имейте в виду, что Redux ориентирован только на управление состоянием (state). В реальном приложении, вы будете использовать UI-биндинги, такие как [react-redux](https://github.com/gaearon/react-redux).

### Высокоуровневые экспорты

* [createStore(reducer, [initialState])](createStore.md)
* [combineReducers(reducers)](combineReducers.md)
* [applyMiddleware(...middlewares)](applyMiddleware.md)
* [bindActionCreators(actionCreators, dispatch)](bindActionCreators.md)
* [compose(...functions)](compose.md)

### API хранилища

* [Store](Store.md)
  * [getState()](Store.md#getState)
  * [dispatch(action)](Store.md#dispatch)
  * [subscribe(listener)](Store.md#subscribe)
  * [replaceReducer(nextReducer)](Store.md#replaceReducer)

### Импорты

Каждая функция описана как экспорт верхнего уровня.
Вы можете импортировать любые из них, например:

#### ES6

```js
import { createStore } from 'redux'
```

#### ES5 (CommonJS)

```js
var createStore = require('redux').createStore
```

#### ES5 (UMD build)

```js
var createStore = Redux.createStore
```
