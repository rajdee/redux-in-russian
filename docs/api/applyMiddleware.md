# `applyMiddleware(...middlewares)`

Мидлвэр (middleware) является предложенным способом расширения Redux с помощью пользовательской функциональности. Мидлвэр позволяет вам обернуть метод хранилища [`dispatch`](Store.md#dispatch) для пользы и дела. Ключевой особенностью мидлвэра является то, что они компонуемы. Несколько мидлвэров могут быть объединены вместе, при этом каждый мидлвэр не требует знания о том, что происходит до или после него в цепочке.

Наиболее распространенным случаем использования мидлвэров является поддержка асинхронных действий без необходимости в большом количестве шаблонного кода или зависимости от библиотек типа [Rx](https://github.com/Reactive-Extensions/RxJS). Это позволяет вам вызывать [асинхронные действия](../Glossary.md#async-action) помимо обычных действий.

Например, [redux-thunk](https://github.com/gaearon/redux-thunk) позволяет генераторам действий инвертировать управление вызывая функции. Они будут получать [`dispatch`](Store.md#dispatch), как аргумент и могут вызывать его асинхронно. Такие функции называются *преобразователями (thunks)*. Другим примером мидлвэра является [redux-promise](https://github.com/acdlite/redux-promise). Он позволяет вам вызывать асинхронное действие c [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) и вызывать обычные действия, когда промис вернет resolve.

Мидлвэры нельзя сравнивать с [`createStore`](createStore.md) и это не фундаментальная часть архитектуры Redux, но, мы считаем, что достаточно полезно поддерживать их прямо в ядре. Таким образом, существует единственный стандартный способ расширить [`dispatch`](Store.md#dispatch) в экосистеме и разные мидлвэры могут конкурировать в выразительности и полезности.

#### Параметры

* `...middlewares` (*arguments*): Функции, которые соответствуют Redux *middleware API*. Каждый мидлвэр получает [`dispatch`](Store.md#dispatch) и [`getState`](Store.md#getState) функции в качестве именованных аргументов и возвращает функцию. Эта функция будет передана `next` dispatch-методу мидлвэра и, как ожидается, вернет функцию `действия`, вызывающую `next(action)` с возможно другим аргументом или позже или, возможно, не вызывая его вообще. Последний мидлвэр в цепочке получит реальный [`dispatch`](Store.md#dispatch) метод хранилища, в качестве `next` параметра, таким образом завершая цепочку. Следовательно сигнатурой мидлвэра является `({ getState, dispatch }) => next => action`.

#### Возвращает

(*Function*) Расширитель хранилища, который применяет полученный мидлвэр. The store enhancer signature is `createStore => createStore'` but the easiest way to apply it is to pass it to [`createStore()`](./createStore.md) as the last `enhancer` argument.

#### Пример: Custom Logger Middleware

```js
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'

function logger({ getState }) {
  return (next) => (action) => {
    console.log('will dispatch', action)

    // Call the next dispatch method in the middleware chain.
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // This will likely be the action itself, unless
    // a middleware further in chain changed it.
    return returnValue
  }
}

let store = createStore(
  todos,
  [ 'Use Redux' ],
  applyMiddleware(logger)
)

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware'
})
// (These lines will be logged by the middleware:)
// will dispatch: { type: 'ADD_TODO', text: 'Understand the middleware' }
// state after dispatch: [ 'Use Redux', 'Understand the middleware' ]
```

#### Example: Using Thunk Middleware for Async Actions

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from './reducers'

let reducer = combineReducers(reducers)
// applyMiddleware supercharges createStore with middleware:
let store = createStore(reducer, applyMiddleware(thunk))

function fetchSecretSauce() {
  return fetch('https://www.google.com/search?q=secret+sauce')
}

// These are the normal action creators you have seen so far.
// The actions they return can be dispatched without any middleware.
// However, they only express “facts” and not the “async flow”.

function makeASandwich(forPerson, secretSauce) {
  return {
    type: 'MAKE_SANDWICH',
    forPerson,
    secretSauce
  }
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: 'APOLOGIZE',
    fromPerson,
    toPerson,
    error
  }
}

function withdrawMoney(amount) {
  return {
    type: 'WITHDRAW',
    amount
  }
}

// Even without middleware, you can dispatch an action:
store.dispatch(withdrawMoney(100))

// But what do you do when you need to start an asynchronous action,
// such as an API call, or a router transition?

// Meet thunks.
// A thunk is a function that returns a function.
// This is a thunk.

function makeASandwichWithSecretSauce(forPerson) {

  // Invert control!
  // Return a function that accepts `dispatch` so we can dispatch later.
  // Thunk middleware knows how to turn thunk async actions into actions.

  return function (dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    )
  }
}

// Thunk middleware lets me dispatch thunk async actions
// as if they were actions!

store.dispatch(
  makeASandwichWithSecretSauce('Me')
)

// It even takes care to return the thunk's return value
// from the dispatch, so I can chain Promises as long as I return them.

store.dispatch(
  makeASandwichWithSecretSauce('My wife')
).then(() => {
  console.log('Done!')
})

// In fact I can write action creators that dispatch
// actions and async actions from other action creators,
// and I can build my control flow with Promises.

function makeSandwichesForEverybody() {
  return function (dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {

      // You don't have to return Promises, but it's a handy convention
      // so the caller can always call .then() on async dispatch result.

      return Promise.resolve()
    }

    // We can dispatch both plain object actions and other thunks,
    // which lets us compose the asynchronous actions in a single flow.

    return dispatch(
      makeASandwichWithSecretSauce('My Grandma')
    ).then(() =>
      Promise.all([
        dispatch(makeASandwichWithSecretSauce('Me')),
        dispatch(makeASandwichWithSecretSauce('My wife'))
      ])
    ).then(() =>
      dispatch(makeASandwichWithSecretSauce('Our kids'))
    ).then(() =>
      dispatch(getState().myMoney > 42 ?
        withdrawMoney(42) :
        apologize('Me', 'The Sandwich Shop')
      )
    )
  }
}

// This is very useful for server side rendering, because I can wait
// until data is available, then synchronously render the app.

import { renderToString } from 'react-dom/server'

store.dispatch(
  makeSandwichesForEverybody()
).then(() =>
  response.send(renderToString(<MyApp store={store} />))
)

// I can also dispatch a thunk async action from a component
// any time its props change to load the missing data.

import { connect } from 'react-redux'
import { Component } from 'react'

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(
      makeASandwichWithSecretSauce(this.props.forPerson)
    )
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(
        makeASandwichWithSecretSauce(nextProps.forPerson)
      )
    }
  }

  render() {
    return <p>{this.props.sandwiches.join('mustard')}</p>
  }
}

export default connect(
  state => ({
    sandwiches: state.sandwiches
  })
)(SandwichShop)
```

#### Советы

* Middleware only wraps the store's [`dispatch`](Store.md#dispatch) function. Technically, anything a middleware can do, you can do manually by wrapping every `dispatch` call, but it's easier to manage this in a single place and define action transformations on the scale of the whole project.

* If you use other store enhancers in addition to `applyMiddleware`, make sure to put `applyMiddleware` before them in the composition chain because the middleware is potentially asynchronous. For example, it should go before [redux-devtools](https://github.com/gaearon/redux-devtools) because otherwise the DevTools won't see the raw actions emitted by the Promise middleware and such.

* If you want to conditionally apply a middleware, make sure to only import it when it's needed:

  ```js
  let middleware = [ a, b ]
  if (process.env.NODE_ENV !== 'production') {
    let c = require('some-debug-middleware')
    let d = require('another-debug-middleware')
    middleware = [ ...middleware, c, d ]
  }

  const store = createStore(
    reducer,
    preloadedState,
    applyMiddleware(...middleware)
  )
  ```

  This makes it easier for bundling tools to cut out unneeded modules and reduces the size of your builds.

* Ever wondered what `applyMiddleware` itself is? It ought to be an extension mechanism more powerful than the middleware itself. Indeed, `applyMiddleware` is an example of the most powerful Redux extension mechanism called [store enhancers](../Glossary.md#store-enhancer). It is highly unlikely you'll ever want to write a store enhancer yourself. Another example of a store enhancer is [redux-devtools](https://github.com/gaearon/redux-devtools). Middleware is less powerful than a store enhancer, but it is easier to write.

* Middleware sounds much more complicated than it really is. The only way to really understand middleware is to see how the existing middleware works, and try to write your own. The function nesting can be intimidating, but most of the middleware you'll find are, in fact, 10-liners, and the nesting and composability is what makes the middleware system powerful.

* To apply multiple store enhancers, you may use [`compose()`](./compose.md).
