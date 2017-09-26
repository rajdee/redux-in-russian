# Асинхронные действия (Async Actions)

В [базовом руководстве](../basics/README.md) мы построили простое приложение — список дел. Оно было совершенно синхронным. Состояние (State) обновлялось сразу же после того, как было сгенерировано и отправлено действие (dispatch action).

В этом руководстве мы будем строить асинхронное приложение, которое будет использовать API Reddit'а для отображения текущих заголовков для выбранного subreddit'a. Так как же асинхронность вписывается в концепцию Redux?

## Действия

Когда Вы вызываете асинхронное API, есть два ключевых момента времени: момент непосредственного вызова и момент получения ответа или timeout'a.

Для каждого из этих моментов обычно может требоваться изменение состояния приложения (application state). Для изменения состояния вы должны запустить (dispatch) нормальное действие, которое будет обработано редюсером синхронно. Обычно для любого API запроса вам понадобится запустить (dispatch) по крайней мере три разных вида действий (actions):

* **Действие, информирующее редюсер о том, что запрос начался.**

  Редюсер может обрабатывать такой вид действия, переключая флаг `isFetching` в состоянии приложения. Именно так UI понимает, что самое время показать лоадер/спиннер.

* **Действие, информирующее редюсер о том, что запрос успешно завершился.**

  Редюсер может обрабатывать такой вид действия, объединяя полученные из запроса данные с тем срезом состояния, которым управляет этот редюсер и сбрасывая флаг `isFetching`.

* **Действие, информирующее редюсер о том, что запрос завершился неудачей.**

  Редюсер может обрабатывать такой вид действия сбрасывая флаг `isFetching`. Также некоторые редюсеры могут хотеть сохранить сообщение об ошибке, чтобы UI мог его отобразить.

Для всего этого вы можете использовать поле `status` в действиях (actions):

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

Или вы можете объявить отдельные типы для таких действий:

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

Выбор использования одного типа действий с флагами или нескольких отдельных типов действий остается за вами. Это соглашение, которое вы должны утвердить с вашей командой. 

Использование нескольких типов действий оставляют меньше места для ошибки, но это не проблема, если вы генерируете действия и редюсеры с помощью таких библиотек, как [redux-actions](https://github.com/acdlite/redux-actions).

Какое бы соглашение вы не выбрали, следуйте ему на протяжении всего приложения.
В этом руководстве мы будем использовать несколько отдельных типов действий.

## Синхронные генераторы действий

Давайте начнем с объявления нескольких синхронных типов и генераторов действий, которые нам понадобятся в нашем приложении. Тут пользователь может выбрать reddit для отображения:

#### `actions.js`

```js
//тип действия
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

//генератор действия
export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```

Также пользователь может нажать кнопку "обновить" для обновления:

```js
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```

Это были действия отвечающие за взаимодействие с пользователем. Также у нас будет и другой тип действий, отвечающий за сетевые запросы. Сейчас мы просто определим их, а чуть позже посмотрим, как посылать такие действия.

Когда нужно будет фетчить посты для какого-нибудь subreddit'a мы будем посылать действие `REQUEST_POSTS`:

```js
export const REQUEST_POSTS = 'REQUEST_POSTS'

export function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```

Важно чтобы это действие было отделено от `SELECT_SUBREDDIT` и `INVALIDATE_SUBREDDIT`. Хотя они, действия, могут происходить одно за другим, но, с возрастанием сложности приложения вам может понадобится фетчить какие-то данные независимо от действий пользователя, например, для предзагрузки самых популярных subreddit'ов или для того, чтобы изредка обновлять устаревшие данные. Вы можете также захотеть фетчить данные в ответ на смену роута, так что не очень мудро связывать обновление данных с каким-то определенным UI-событием.

Наконец, когда сетевой запрос будет осуществлен, мы отправим действие `RECEIVE_POSTS`:

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

export function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

Это все, что нам нужно знать на текущий момент. Конкретный механизм работы этих действий вместе с сетевыми запросами будет обсуждаться позже.

>##### Обратите внимание на обработчик ошибки

> В реальном приложении вам бы также понадобилось отправлять действие в случае завершения сетевого запроса ошибкой. В этом руководстве мы не будем реализовывать обработку ошибок, но [пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует один из возможных путей ее реализации.

## Разработка структуры хранилища

Как и в базовом руководстве, вам нужно [разработать структуру состояния приложения](../basics/Reducers.md#designing-the-state-shape), прежде чем начинать писать само приложение. В случае с асинхронным кодом появляется больше состояний, о которых нужно позаботиться, так что нам нужно все как следует обдумать.

Часто именно эта часть сбивает с толку новичков, потому что сразу не ясно, какая информация описывает состояние в асинхронном приложении и как организовать все это в одно дерево состояния.

Мы начнем с наиболее общего варианта использования, списков. Веб-приложения часто отображают списки чего-либо. Например, список постов или список друзей. Вам нужно будет решить, какие типы списков сможет отображать ваше приложение. Вам нужно хранить их отдельно в состоянии, потому что в этом случае вы можете кешировать их и снова обновлять данные при необходимости. 

Вот как может выглядеть структура состояния для нашего  приложения “Reddit headlines”:

```js
{
  selectedSubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```

тут есть несколько важных моментов:

* Мы храним информацию о каждом subreddit’е отдельно, следовательно мы можем кешировать любой subreddit. Когда пользователь переключается между ними во второй раз, обновление UI будет мгновенным и сможем не перезагружать данные если мы этого не хотим. Не переживайте о том, что все эти элементы (subreddit'ы, а их может быть очень много) будут находиться в памяти: Вам не понадобятся никакие чистки памяти, если только вы и ваш пользователь не имеете дело с десятками тысяч элементов и при этом пользователь очень редко закрывает вкладку браузера.

* Для каждого списка элементов вы захотите хранить `isFetching` для показа спиннера, `didInvalidate`, который вы потом сможете установить в true, если данные устареют, `lastUpdated` для того чтобы знать, когда данные были обновлены в последний раз, и собственно `items`. В реальном приложении вы также захотите хранить состояние страничной навигации: `fetchedPageCount` и `nextPageUrl`.

>##### Обратите внимание на вложенные сущности

> В этом примере мы храним полученные элементы вместе с информацией о постраничной навигации. Однако этот подход будет очень плох, если у вас будут вложенные сущности, которые ссылаются друг на друга или если Вы дадите пользователю возможность редактировать элементы. Представьте, что пользователь хочет отредактировать загруженный пост, но этот пост сдублирован в нескольких местах в дереве состояния (state tree). Реализация такого будет очень болезненна.

>Если у вас есть вложенные сущности или если вы даете возможность редактировать загруженные элементы, то вы должны хранить их отдельно в состоянии, как если бы оно (состояние) было базой данных. И в информации о постраничной навигации вы можете ссылаться на такие элементы только по их ID. Это позволит вам всегда держать их в актуальном состоянии. [Пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует этот подход вместе с [normalizr](https://github.com/gaearon/normalizr) для упорядочивания вложенных ответов API. С таким подходом ваше состояние (state) может выглядеть вот так:

>```js
> {
>   selectedSubreddit: 'frontend',
>   entities: {
>     users: {
>       2: {
>         id: 2,
>         name: 'Andrew'
>       }
>     },
>     posts: {
>       42: {
>         id: 42,
>         title: 'Confusion about Flux and Relay',
>         author: 2
>       },
>       100: {
>         id: 100,
>         title: 'Creating a Simple Application Using React JS and Flux Architecture',
>         author: 2
>       }
>     }
>   },
>   postsBySubreddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [ 42, 100 ]
>     }
>   }
> }
>```

>В этом руководстве мы не будем упорядочивать сущности, но это то, о чем вам стоит задуматься для более динамичного приложения.

## Обработка действий

Перед тем как переходить к деталям отправки действий вместе с сетевыми запросами, мы напишем редюсеры для действий, которые определили выше.

>##### Обратите внимание на компоновку редюсеров

> Предполагается, что вы понимаете что такое компоновка редюсеров с помощью функции [`combineReducers()`](../api/combineReducers.md), описанной в разделе [Разделение редюсеров](../basics/Reducers.md#splitting-reducers) в [базовом руководстве](../basics/README.md). Если это не так, то, пожалуйста, [сначала прочтите это](../basics/Reducers.md#splitting-reducers).

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedSubreddit(state = 'reactjs', action) {
  switch (action.type) {
    case SELECT_SUBREDDIT:
      return action.subreddit
    default:
      return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = {}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedSubreddit
})

export default rootReducer
```

Две части этого кода вызывают особый интерес:

* Мы используем ES6-синтаксис вычисляемого свойства, т.е. мы можем обновить `state[action.subreddit]` с помощью `Object.assign()` с использованием меньшего количества строк кода. Вот это:

  ```js
  return Object.assign({}, state, {
    [action.subreddit]: posts(state[action.subreddit], action)
  })
  ```
  эквивалентно этому:

  ```js
  let nextState = {}
  nextState[action.subreddit] = posts(state[action.subreddit], action)
  return Object.assign({}, state, nextState)
  ```
* Мы извлекли `posts(state, action)`, который управляет состоянием конкретного списка постов. Это просто [компоновка редюсеров](../basics/Reducers.md#splitting-reducers)! Нам выбирать, как разбить/разделить редюсер на более мелкие редюсеры и в этом случае, мы доверяем обновление элементов внутри объекта функции-редюсеру `posts`. [Пример из реальной жизни](../introduction/Examples.html#real-world) идет еще дальше, показывая, как создавать фабрику редюсеров для параметризированных редюсеров постраничной навигации.

Помните, что редюсеры — это всего лишь функции, т.е. вы можете использовать функциональную композицию (речь о функциональном подходе к программированию и композиции функций *прим. переводчика*) и функции высшего порядка так часто, как вам это будет удобно.

## Асинхронные генераторы действий

Наконец, как мы используем синхронные генераторы действий, [созданные нами ранее](#synchronous-action-creators) вместе с сетевыми запросами? Стандартный для Redux путь — это использование [Redux Thunk middleware](https://github.com/gaearon/redux-thunk). Этот мидлвэр (middleware) содержится в отдельном пакете, который называется `redux-thunk`. Мы поясним, как работают мидлвэры [позже](Middleware.md). Сейчас есть одна важная вещь, о которой вам нужно знать: при использовании конкретно этого мидлвэра, генератор действий может вернуть функцию, вместо объекта действия. Таким образом, генератор действия превращается в [преобразователь (Thunk)](https://en.wikipedia.org/wiki/Thunk)

Когда генератор действия вернет функцию, эта функция будет вызвана мидлвэром Redux Thunk. Этой функции не обязательно быть чистой. Таким образом, в ней разрешается инициировать побочные эффекты (side effects), в том числе и асинхронные вызовы API. Также эти функции могут вызывать действия, такие же синхронные действия, которые мы отправляли ранее.

Мы все еще можем определить эти специальные thunk-генераторы действий внутри нашего `actions.js` файла:

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

// Тут мы встречаемся с нашим первым thunk-генератором действий! Хотя его содержимое
// отличается, вы должны использовать его, как и любой другой генератор действий:
// store.dispatch(fetchPosts('reactjs'))

export function fetchPosts(subreddit) {

  // Thunk middleware знает, как обращаться с функциями.
  // Он передает метод действия в качестве аргумента функции,
  // т.к. это позволяет отправить действие самостоятельно.

  return function (dispatch) {

    // Первая отправка: состояние приложения обновлено, 
    // чтобы сообщить, что запускается вызов API.

    dispatch(requestPosts(subreddit))

    // Функция, вызываемая Thunk middleware, может возвращать значение, 
    // которое передается как возвращаемое значение метода dispatch.

    // В этом случае мы возвращаем Promise.
    // Thunk middleware не требует этого, но это удобно для нас.

    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // Мы можем вызывать dispatch много раз!
        // Здесь мы обновляем состояние приложения с результатами вызова API.

        dispatch(receivePosts(subreddit, json))
      )

      // В реальном приложении вы также захотите ловить ошибки сетевых запросов.
      
  }
}
```

>##### Примечание по `fetch`

>Мы используем [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) в примерах. Это новое API для создания сетевых запросов, которое заменяет `XMLHttpRequest` в большинстве стандартных случаев. Поскольку большинство браузеров до сих пор не поддерживают его нативно, мы полагаем, что вы для этого используете[`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) библиотеку:

>```js
// Добавьте это в каждый файл, где вы используете `fetch`
>import fetch from 'isomorphic-fetch'
>```

>Внутри она использует [полифил `whatwg-fetch`](https://github.com/github/fetch) на клиенте и [`node-fetch`](https://github.com/bitinn/node-fetch) на сервере, поэтому вам не понадобится менять вызовы API, если вы захотите сделать ваше приложение [универсальным](https://medium.com/@mjackson/universal-javascript-4761051b7ae9).

>Помните, что любой полифил `fetch` предполагает, что полифил [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) уже присутствует. Самый простой способ убедиться, что вы подключили Promise-полифил — это подключить ES6-полифил Babel во входной точке, прежде чем любой другой код запустится:

>```js
>// Добавьте это в самом начале вашего приложения
>import 'babel-core/polyfill'
>```

Как мы добавляем мидлвэр Redux Thunk в механизм диспетчера? Для этого мы используем метод [`applyMiddleware()`](../api/applyMiddleware.md) из Redux, как показано ниже:

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk'
import { createLogger } from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // позволяет нам отправлять функции
    loggerMiddleware // аккуратно логируем действия
  )
)

store.dispatch(selectSubreddit('reactjs'))
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```

Хорошая новость о преобразователях: они могут направлять результаты друг другу.

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // Помните, что функция также получает getState(),
  // который позволяет вам выбрать, что отправить дальше.

  // Это полезно для того, чтобы избежать сетевого запроса,
  // если доступно закешированное значение.

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // Обратимся у thunk из thunk!
      return dispatch(fetchPosts(subreddit))
    } else {
      // Дадим вызвавшему коду знать, что ждать нечего.
      return Promise.resolve()
    }
  }
}
```

Это позволяет нам писать более сложный поток асинхронного управления постепенно, в то время, как потребляющий код может оставаться таким же довольно долгое время:

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState())
)
```

>##### Примечание о серверном рендеринге

> Асинхронные генераторы действий особенно удобны для серверного рендеринга. Вы можете создать хранилище, вызвать отдельный асинхронный генератор действия, который вызовет другие асинхронные генераторы действия для выборки данных для всей части вашего приложения и отрендерит, только после того, как promise его вернет. Затем ваше хранилище будет полностью гидратировано с состоянием, необходимым для рендеринга.

[Thunk middleware](https://github.com/gaearon/redux-thunk) — это не единственный путь управления асинхронными действиями в Redux. Вы можете использовать [redux-promise](https://github.com/acdlite/redux-promise) или [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) для отправки Promises вместо функций. Вы можете отправлять Наблюдателей (Observables) с [redux-rx](https://github.com/acdlite/redux-rx). Вы даже можете писать собственные мидлвэры, для описания вызовов вашего API, как например в [real world example](../introduction/Examples.html#real-world). Решать вам, попробовать несколько вариантов, выбрать конвенции, которые вам нравятся и следовать им, будь то с использованием мидлвэра или без него.

## Подключение к UI

Отправка асинхронных действий не отличается от отправки синхронных действий, поэтому мы не будем обсуждать это в деталях. Почитайте [использование в связке с React](../basics/UsageWithReact.md) для знакомства с использованием Redux с React-компонентами. С полным исходным кодом, обсуждаемым в этом примере, вы можете ознакомится в [примере Reddit API](ExampleRedditAPI.md)

## Следующие шаги

Прочтите [Асинхронный поток](AsyncFlow.md) для резюмирования того, как асинхронные действия вписываются в поток Redux.
