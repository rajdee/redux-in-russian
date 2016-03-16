# Глоссарий

Это глоссарий основных терминов в Redux, наряду с их сигнатурами типа. Типы описаны при помощи [Flow notation](http://flowtype.org/docs/quick-reference.html).

## Состояние (State)

```js
type State = any;
```

*Состояние* (также *дерево состояния*) — широкое понятие, но в Redux API это, как правило, отсылка к единственному состоянию, которое управляется хранилищем (store) и возвращается [`getState()`](api/Store.md#getState). Оно представляет собой все состояние Redux приложения, которое обычно является объектом с глубокой вложенностью.

Как правило, состояние верхнего уровня — это объект или какая-то другая коллекция вида ключ-значение, например Map, но технически это может быть любой тип. Вместе с тем, вам нужно стараться поддерживать состояние сериализуемым. Не кладите внутрь ничего, что потом не сможете легко превратить в JSON.

## Действие (Action)

```js
type Action = Object;
```

*Действие* — это простой объект, который представляет намерение изменить состояние. Действия — единственный путь получить данные в хранилище. Любые данные, будь то события UI, коллбэки сетевых запросов или любые другие ресурсы как веб-сокеты, должны быть в итоге обработаны, как действия.

Действия обязаны иметь поле `type`, которое указывает тип производимого действия. Типы также могут быть определены как константы и импортированы из другого модуля. Лучше использовать строки для `type`, чем [Символы](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol), потому что строки сериализуемы.

Вся остальная структура, кроме `type`, полностью на ваше усмотрение. Если вы заинтересованы, посмотрите [Flux Standard Action](https://github.com/acdlite/flux-standard-action) для рекомендаций как нужно создавать действия.

Также смотрите [асинхронное действие](#async-action) ниже.

## Редьюсер (Reducer)

```js
type Reducer<S, A> = (state: S, action: A) => S;
```

A *reducer* (also called a *reducing function*) is a function that accepts an accumulation and a value and returns a new accumulation. They are used to reduce a collection of values down to a single value.

Reducers are not unique to Redux—they are a fundamental concept in functional programming.  Even most non-functional languages, like JavaScript, have a built-in API for reducing. In JavaScript, it's [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce).

In Redux, the accumulated value is the state object, and the values being accumulated are actions. Reducers calculate a new state given the previous state and an action. They must be *pure functions*—functions that return the exact same output for given inputs. They should also be free of side-effects. This is what enables exciting features like hot reloading and time travel.

Reducers are the most important concept in Redux.

*Do not put API calls into reducers.*

## Dispatching Function

```js
type BaseDispatch = (a: Action) => Action;
type Dispatch = (a: Action | AsyncAction) => any;
```

A *dispatching function* (or simply *dispatch function*) is a function that accepts an action or an [async action](#async-action); it then may or may not dispatch one or more actions to the store.

We must distinguish between dispatching functions in general and the base [`dispatch`](api/Store.md#dispatch) function provided by the store instance without any middleware.

The base dispatch function *always* synchronously sends an action to the store’s reducer, along with the previous state returned by the store, to calculate a new state. It expects actions to be plain objects ready to be consumed by the reducer.

[Middleware](#middleware) wraps the base dispatch function. It allows the dispatch function to handle [async actions](#async-action) in addition to actions. Middleware may transform, delay, ignore, or otherwise interpret actions or async actions before passing them to the next middleware. See below for more information.

## Action Creator

```js
type ActionCreator = (...args: any) => Action | AsyncAction;
```

An *action creator* is, quite simply, a function that creates an action. Do not confuse the two terms—again, an action is a payload of information, and an action creator is a factory that creates an action.

Calling an action creator only produces an action, but does not dispatch it. You need to call the store’s [`dispatch`](api/Store.md#dispatch) function to actually cause the mutation. Sometimes we say *bound action creators* to mean functions that call an action creator and immediately dispatch its result to a specific store instance.

If an action creator needs to read the current state, perform an API call, or cause a side effect, like a routing transition, it should return an [async action](#async-action) instead of an action.

## Async Action

```js
type AsyncAction = any;
```

An *async action* is a value that is sent to a dispatching function, but is not yet ready for consumption by the reducer. It will be transformed by [middleware](#middleware) into an action (or a series of actions) before being sent to the base [`dispatch()`](api/Store.md#dispatch) function. Async actions may have different types, depending on the middleware you use. They are often asynchronous primitives, like a Promise or a thunk, which are not passed to the reducer immediately, but trigger action dispatches once an operation has completed.

## Посредник (Middleware)

```js
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State };
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch;
```

Посредник — это функция высшего порядка, которая создает [функцию отправки (dispatch function)](#dispatching-function), возвращающую новую функцию отправки. Она часто возвращает [асинхронное действие](#async-action) в действия.

Посредники компонуемы с помощью функции композиции. Это полезно для регистрации действий, выполнения побочных действий (side effects), таких как маршрутизация или превращения асинхронного вызова API в серию синхронных действий

См. [`applyMiddleware(...middlewares)`](./api/applyMiddleware.md) для более детальной информации по посредникам.

## Хранилище (Store)

```js
type Store = {
  dispatch: Dispatch;
  getState: () => State;
  subscribe: (listener: () => void) => () => void;
  replaceReducer: (reducer: Reducer) => void;
};
```

Хранилище — это объект, который хранит дерево состояний приложения.
В приложении должно быть только одно хранилище, так построение происходит на уровне преобразователя (reducer).
 
- [`dispatch(action)`](api/Store.md#dispatch) базовая функция отправки (dispatch), описанная выше.
- [`getState()`](api/Store.md#getState) возвращает текущее состояние хранилища.
- [`subscribe(listener)`](api/Store.md#subscribe) регистрирует функцию, которая будет вызвана при изменении состояния.
- [`replaceReducer(nextReducer)`](api/Store.md#replaceReducer) может быть использован для реализации горячей перезагрузки (hot reload) и разделения кода. Скорее всего Вы не будете использовать ee.

См. [store API reference](api/Store.md#dispatch) для получения дополнительной информации.

## Генератор хранилища (Store creator)

```js
type StoreCreator = (reducer: Reducer, initialState: ?State) => Store;
```

Генератор хранилища — это функция, которая создает хранилище Redux. Как и в случаи с отправляющей функцией, мы должны различать базовый генератор хранилища, [`createStore(reducer, initialState)`](api/createStore.md) экспортирумый из Redux, от генератора хранилища, возвращаемого из расширителей хранилища (store enhancers).

## Расширитель хранилища (Store enhancer)

```js
type StoreEnhancer = (next: StoreCreator) => StoreCreator;
```

Расширитель хранилища — это функция высшего порядка, которая создает генератор хранилища, возвращающий новый, расширенный генератор хранилища. Это похоже на посредник (middleware) тем, что позволяет вам изменять интерфейс в композиционном стиле.

Расширители хранилища аналогичны понятию - "компоненты высшего порядка" в React, которые также иногда называются "усилителями компонент".

Поскольку хранилище является не инстансом, а скорее объектом-коллекцией функций, то копии могут быть запросто созданы и модифицированы, без изменения оригинального хранилища. Пример в описании [`compose`](api/compose.md) демонстрирует это.

Скорее всего, вы никогда не будете писать расширитель хранилища, но вы можете использовать один предоставленный [developer tools](https://github.com/gaearon/redux-devtools). Это то, что делает "путешествие во времени" (time travel) возможным без информирования приложения, о том, что происходит. Занятно, что [реализация Redux посредников](api/applyMiddleware.md) сама по себе является расширителем хранилища.
