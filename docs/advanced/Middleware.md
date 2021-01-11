# Mидлвар (Middleware)

Вы видели мидлвары в действии в [примере асинхронных экшенов](../advanced/AsyncActions.md). Если вы когда-либо использовали такие серверные библиотеки, как [Express](http://expressjs.com/) и [Koa](http://koajs.com/), то, вероятно, вы уже хорошо знакомы с концепцией _мидлвар_. В этих фреймворках мидлвары — это части кода, которые вы можете поместить между фреймворком, принимающим запрос и фреймворком, генерирующим ответ. Например, мидлвары из Express или Koa могут добавлять CORS-заголовки, логирование, сжатие и т.д. Лучшая особенность мидлваров заключается в том, что их можно соединять в цепочки/последовательности. Вы можете использовать множество независимых сторонних мидлваров в одном проекте.

Redux-мидлвары, в отличие от мидлваров Express или Koa, решают немного другие проблемы, но концептуально схожим способом. **Они предоставляют стороннюю точку расширения, между отправкой экшена и моментом, когда этот экшен достигает редьюсера.** Люди используют Redux-мидлвары для логирования, сообщения об ошибках, общения с асинхронным API, роутинга и т.д.

Эта статья разделена на углубленное введение, которое поможет вам хорошо разобраться в концепции, и [пару практических примеров](#Семь-примеров) в самом конце, которые покажут вам всю силу мидлваров. Вам может показаться полезным периодическое переключение между этими частями, так же, как между скукой и вдохновением.

## Понимание мидлваров

Т.к. мидлвары могут использоваться для различных задач, в том числе и для асинхронных обращений к API, то очень важно, чтобы вы понимали, откуда они пришли. Мы покажем вам ход мыслей, шаг за шагом ведущий к мидлварам, используя логирование и сообщения об ошибках в качестве примера.

### Проблема: логирование

Одно из достоинств Redux — он делает изменения состояния приложения предсказуемыми и прозрачными. Каждый раз, когда посылается экшен,  новое состояние вычисляется и сохраняется. Состояние не может измениться самостоятельно, оно может меняться только, как последовательность определенных экшенов.

Разве не было бы хорошо, если бы мы записывали каждое действие, которое происходило в приложении, вместе с состоянием, которое было вычислено после этого действия? Когда что-то идет не так, мы можем просмотреть наш лог и понять, какой именно экшен испортил наше состояние.

<img src='http://i.imgur.com/BjGBlES.png' width='70%'>

Как мы подходим к этому с Redux?

### Попытка #1: Логируем вручную

Простейшее решение - самостоятельно записывать экшен и состояние каждый раз, когда вы вызываете [`store.dispatch(action)`](../api/Store.md#dispatch). На самом деле это не слишком хорошее решение, это просто первый шаг на пути к пониманию проблемы.

>##### Обратите внимание

>Если вы используете [react-redux](https://github.com/gaearon/react-redux) или похожий биндинг,  у вас, скорее всего, не будет прямого доступа к экземпляру стора в ваших компонентах. Для следующих нескольких параграфов представьте, что вы передаете состояние явно.

Например, вы вызываете такой код, когда создаете todo-элемент:

```js
store.dispatch(addTodo('Use Redux'))
```

Для того чтобы логировать экшен и состояние, вы можете изменить код примерно так:

```js
let action = addTodo('Use Redux')

console.log('dispatching', action)
store.dispatch(action)
console.log('next state', store.getState())
```

Это даст желаемый эффект, но вы бы не хотели делать так каждый раз.

### Попытка #2: Оборачиваем Dispatch

Вы можете вынести логирование в функцию:

```js
function dispatchAndLog(store, action) {
  console.log('dispatching', action)
  store.dispatch(action)
  console.log('next state', store.getState())
}
```

Вы можете использовать ее везде вместо обычного `store.dispatch()`:

```js
dispatchAndLog(store, addTodo('Use Redux'))
```

Мы бы могли закончить на этом, но не очень удобно импортировать специальную функцию каждый раз.

### Попытка #3: [Monkeypatching](https://ru.wikipedia.org/wiki/Monkey_patch) для Dispatch

Что, если мы просто заменим функцию `dispatch` в экземпляре стора? Redux стор — это простой объект с [парой методов](../api/Store.md), а мы пишем на JavaScript, следовательно, мы можем применить технику monkeypatch для реализации `dispatch`:

```js
const next = store.dispatch
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

Это уже ближе к тому, что нам нужно! Не важно, откуда мы посылаем экшен, он гарантированно будет залогирован. Monkeypatching никогда не покажется правильным ходом, но пока мы можем с этим жить.

### Проблема: Сообщения об ошибках.

Что, если мы захотим применить **больше одного такого** преобразования к `dispatch`?

Другое такое изменение, которое приходит мне в голову, это сообщения о JavaScript-ошибках в продакшене. Глобальное событие `window.onerror` не надежно потому, что оно в некоторых старых браузерах не предоставляет информацию о стеке вызовов, которая важна для понимания того, почему же произошла ошибка.

Разве не было бы полезно, если бы каждый раз, когда ошибка выбрасывалась, как результат отправки какого-либо экшена, мы могли бы отправить ее (ошибку), вместе со стеком вызовов, экшеном, который вызвал ошибку и актуальным состоянием в сервис сообщения об ошибках, такой как [Sentry](https://getsentry.com/welcome/). В таком случае гораздо легче воспроизвести ошибку в разработке.

Однако важно, чтобы мы держали логирование и сообщения об ошибках раздельно. В идеальном случае, мы хотим получить их, как разные модули из разных пакетов. В противном случае, мы не сможем иметь экосистему из такого рода утилит. (Подсказка: мы медленно подходим к тому, что такое мидлвары!)

Если логирование и сообщения об ошибках являются отдельными утилитами, то они могут выглядеть так:

```js
function patchStoreToAddLogging(store) {
  const next = store.dispatch
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}

function patchStoreToAddCrashReporting(store) {
  const next = store.dispatch
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action)
    } catch (err) {
      console.error('Caught an exception!', err)
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      })
      throw err
    }
  }
}
```

Если эти функции опубликованы, как отдельные модули, то позже мы можем использовать их для изменения нашего стора:

```js
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

Но это все еще не очень хорошо.

### Попытка #4: Прячем Monkeypatching

Monkeypatching — это хак. "Замените любой метод, который хотите" — что это за вид API? Давайте разберемся в его сути. Ранее наши функции заменяли `store.dispatch`. Что если бы они вместо этого *возвращали* новую функцию `dispatch`?

```js
function logger(store) {
  let next = store.dispatch

  // ранее было так:
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

Мы могли бы предоставить функцию-помощник внутри Redux, которая могла бы применять актуальный monkeypatching, как часть имплементации:

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  // Изменяем функцию dispatch каждым мидлваром.
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  )
}
```

Мы можем использовать такой подход для применения нескольких мидлваров:

```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```

Тем не менее - это все еще monkeypatching. Факт того, что мы прячем его внутри библиотеки, не отменяет использования monkeypatching.

### Попытка #5: Убираем Monkeypatching

Зачем мы перезаписываем `dispatch`? Конечно же для того, чтобы иметь возможность потом его вызвать. Но есть еще и другая причина: каждый мидлвар имеет доступ (и возможность вызвать) ранее обернутый `store.dispatch`:

```js
function logger(store) {
  // Обязательно нужно закешировать функцию, которую вернул предыдущий мидлвар:
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

Это важно для возможности объединять мидлвары в цепочки!

Если `applyMiddlewareByMonkeypatching` не сохранит `store.dispatch` сразу после обработки первого мидлвара, `store.dispatch` будет продолжать ссылаться на оригинальную функцию `dispatch`. Следовательно второй мидлвар тоже будет связан с оригинальной функцией `dispatch`.

Но есть еще другой метод реализации объединения мидлваров в цепочки (chaining). Мидлвар мог бы принимать функцию отправки экшена `next()` в параметрах вместо того, чтобы читать ее из экземпляра стора.

```js
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action)
      let result = next(action)
      console.log('next state', store.getState())
      return result
    }
  }
}
```

Это тот момент, когда [“we need to go deeper”](http://knowyourmeme.com/memes/we-need-to-go-deeper), так что имеет смысл потратить некоторе время на это. Каскад функций выглядит пугающим. Стрелочные функции из ES6 делают это [каррирование](https://en.wikipedia.org/wiki/Currying) чуть более простым для глаз:

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

**Именно так выглядят мидлвары в Redux.**

Теперь мидлвар принимает функцию отправки экшена `next()` и возвращает другую функцию отправки экшена, которая, в свою очередь, является функцией отправки экшена `next()` для мидлвара слева. Все еще полезно иметь доступ к некоторым методам стора, например к `getState()`, следовательно, `store` остается доступен, как аргумент самого верхнего уровня.


### Попытка #6: Простейшее применение мидлваров

Вместо `applyMiddlewareByMonkeypatching()` мы могли бы написать функцию `applyMiddleware()`, которая сначала получает финальную, полностью обернутую функцию `dispatch()` и возвращает копию стора, которая использует эту функцию:

```js
// Осторожно: Простейшая имплементация!
// Это *не* Redux API.
function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()
  let dispatch = store.dispatch
  middlewares.forEach(middleware => (dispatch = middleware(store)(dispatch)))
  return Object.assign({}, store, { dispatch })
}
```

Реализация [`applyMiddleware()`](../api/applyMiddleware.md), которая поставляется с Redux, похожа на эту, но **отличается тремя важными аспектами**:

* Она предоставляет мидлвару только подмножество [API стора](../api/Store.md): методы [`dispatch(action)`](../api/Store.md#dispatch) и [`getState()`](../api/Store.md#getState).

* Она использует некоторые хитрости для того, чтобы убедиться, что, экшен снова пройдет через всю цепочку мидлваров, включая текущий, если вы вызываете `store.dispatch(action)` из вашего мидлвара вместо `next(action)`. Это полезно для асинхронных мидлваров, как мы [ранее](AsyncActions.md) видели.

* Для того чтобы гарантировать, что вы можете применить мидлвар только один раз, она работает с `createStore()`, а не с самим `store`. Вместо `(store, middlewares) => store`, ее сигнатурой является `(...middlewares) => (createStore) => createStore`.

#### Предостережение: отправка во время установки

Пока `applyMiddleware` выполняет и настраивает ваши мидлвари, функция` store.dispatch` будет указывать на оригинальную версию, предоставляемую `createStore`. Отправка приведет к тому, что никакой другой мидлвар не будет применен. Если вы ожидаете взаимодействия с другим мидлваром во время настройки, вы, вероятно, будете разочарованы. Из-за этого неожиданного поведения `applyMiddleware` выдаст ошибку, если вы попытаетесь отправить экшен до завершения установки. Вместо этого вам следует, либо напрямую связываться с этим другим мидлваром через общий объект (для мидлвара, вызывающего API, это может быть ваш клиентский объект API), либо ждать, пока мидлвар не будет создан с колбэком.

### Финальный подход

Дан мидлвар который мы только что написали:

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

Вот так можно его применить к Redux стору:

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

const todoApp = combineReducers(reducers)
const store = createStore(
  todoApp,
  // applyMiddleware() tells createStore() how to handle middleware
  applyMiddleware(logger, crashReporter)
)
```

Вот и все! Теперь любые экшены, отправленные в экземпляр стора, будут проходить через `logger` и `crashReporter`:

```js
// будет проходить через `logger` и `crashReporter`!
store.dispatch(addTodo('Use Redux'))
```

## Семь примеров

Если ваша голова вскипела от прочтения предыдущего раздела, представьте, каково было написать это. Этот раздел предназначен для расслабления меня и вас и поможет запустить ваши шестеренки.

Каждая из функций, приведенных ниже, является валидным Redux-мидлваром. Они не являются в равной степени полезными, но, по крайней мере, они в равной степени забавны.

```js
/**
 * Логирует все экшены и состояния после того, как они будут отправлены.
 */
const logger = store => next => action => {
  console.group(action.type)
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  console.groupEnd(action.type)
  return result
}

/**
 * Отправляет отчеты об ошибках когда обновляется состояние и уведомляются слушатели.
 */
const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}

/**
 * Планирует экшены с { meta: { delay: N } }, которые будут отложены на N миллисекунд.
 * Создает `dispatch`, возвращающий функцию, для отмены таймаута.
 */
const timeoutScheduler = store => next => action => {
  if (!action.meta || !action.meta.delay) {
    return next(action)
  }

  let timeoutId = setTimeout(
    () => next(action),
    action.meta.delay
  )

  return function cancel() {
    clearTimeout(timeoutId)
  }
}

/**
 * Планирует экшены с { meta: { raf: true } }, которые будут отправлены внутри 
 * фрейма rAF цикла. Создает  `dispatch`, который возвращает функцию для удаления 
 * экшена из очереди.
 */
const rafScheduler = store => next => {
  const queuedActions = []
  let frame = null

  function loop() {
    frame = null
    try {
      if (queuedActions.length) {
        next(queuedActions.shift())
      }
    } finally {
      maybeRaf()
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop)
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action)
    }

    queuedActions.push(action)
    maybeRaf()

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    }
  }
}

/**
 * Позволяет вам отправлять промисы в дополнение к экшенам.
 * Если промис зарезолвен, его результат будет отправлен как экшен.
 * Промис возвращается из `dispatch`, т.о. вызывающая функция может 
 * обрабатывать отказ (rejection) промиса.
 */
const vanillaPromise = store => next => action => {
  if (typeof action.then !== 'function') {
    return next(action)
  }

  return Promise.resolve(action).then(store.dispatch)
}

/**
 * Позволяет вам отправлять специальные экшены с полем { promise }.
 * Этот мидлвар превратит их в единственный экшен в начале,
 * и в единственный успешный (или неудачный) экшен, когда `promise` будет зарезолвен.
 *
 * Для удобства `dispatch` будет возвращать промис, т.е. вызывающая функция 
 * может ожидать разрешения этого промиса.
 */
const readyStatePromise = store => next => action => {
  if (!action.promise) {
    return next(action)
  }

  function makeAction(ready, data) {
    const newAction = Object.assign({}, action, { ready }, data)
    delete newAction.promise
    return newAction
  }

  next(makeAction(false))
  return action.promise.then(
    result => next(makeAction(true, { result })),
    error => next(makeAction(true, { error }))
  )
}

/**
 * Позволяет вам отправлять функцию вместо экшена.
 * Функция будет принимать `dispatch` и `getState` в качестве аргументов.
 *
 * Полезно для раннего выхода (условия над `getState()`), а также для 
 * асинхронного потока управления (может `dispatch()` что-то другое)
 * 
 * `dispatch` будет возвращать значение отправляемой функции.
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)

// Вы можете использовать их все! (Это не значит, что вы должны.)
const todoApp = combineReducers(reducers)
const store = createStore(
  todoApp,
  applyMiddleware(
    rafScheduler,
    timeoutScheduler,
    thunk,
    vanillaPromise,
    readyStatePromise,
    logger,
    crashReporter
  )
)
```
