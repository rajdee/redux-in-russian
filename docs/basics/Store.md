# Хранилище (Store)

В предыдущих разделах мы определили [действия (actions)](Actions.md), которые представляют факт того, что "что-то случилось" и [редюсеры](Reducers.md), которые обновляют состояние (state) в соответствии с этими действиями. 

**Хранилище (Store)** — это объект, который соединяет эти части вместе. Хранилище берет на себя следующие задачи:

* содержит состояние приложения (application state);
* предоставляет доступ к состоянию с помощью [`getState()`](../api/Store.md#getState);
* предоставляет возможность обновления состояния с помощью [`dispatch(action)`](../api/Store.md#dispatch);
* регистрирует слушателей (listeners) c помощью [`subscribe(listener)`](../api/Store.md#subscribe).

Важно отметить, что у Вас будет только одно хранилище в Redux-приложении. Если Вы захотите разделить логику обработки данных, то нужно будет использовать [компоновку редюсеров (reducer composition)](Reducers.md#splitting-reducers) вместо использования множества хранилищ (stores).

Очень легко создать Хранилище (Store), если у Вас есть редюсер. В [предыдущем разделе](Reducers.md) мы использовали [`combineReducers()`](../api/combineReducers.md) для комбинирования редюсеров в один глобальный редюсер. Теперь мы их импортируем и передадим в [`createStore()`](../api/createStore.md).

```js
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

Вы можете объявить начальное состояние, передав его вторым аргументом в [`createStore()`](../api/createStore.md). Это полезно для пробрасывания состояния на клиент из состояния приложения Redux, работающего на сервере, когда вы пишете универсальное приложение.

```js
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## Отправка действий (Dispatching Actions)

На текущий момент у нас есть созданное хранилище, давайте проверим, как работает наше приложение! Даже без UI-части мы уже можем проверить логику обновления состояния.

```js
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// Выведем в консоль начальное состояние
console.log(store.getState())

// Каждый раз при обновлении состояния - выводим его
// Отметим, что subscribe() возвращает функцию для отмены регистрации слушателя
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// Отправим несколько действий
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// Прекратим слушать обновление состояния
unsubscribe()
```

Вы можете видеть, как выполнение кода, приведенного выше, меняет состояние, содержащееся в хранилище:

<img src='http://i.imgur.com/zMMtoMz.png' width='70%'>

Мы смогли определить поведение нашего приложения даже до того, как начали создавать какой-то UI. Мы не будем делать этого в этом руководстве, но с этого момента Вы можете писать тесты для редюсеров и генераторов действий (action creators). Вам не нужно будет ничего "мокать", потому что редюсеры — это просто функции. Вызывайте их и делайте проверки (make assertions) того, что они возвращают. 

## Исходный код (Source Code)

#### `index.js`

```js
import { createStore } from 'redux'
import todoApp from './reducers'

let store = createStore(todoApp)
```

## Следующие шаги

Перед тем как создавать UI для нашего todo-приложения, мы сделаем небольшую прогулку, чтобы посмотреть, [что такое поток данных (data flows) в Redux-приложении](DataFlow.md).
