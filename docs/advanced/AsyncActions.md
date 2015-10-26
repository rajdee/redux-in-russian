# Асинхронные действия (Async Actions)

В [базовом руководстве](../basics/README.md) мы построили простое приложение - список дел. Оно было совершенно синхронным. Состояние (State) обновлялось сразу же после того, как было сгенерировано и отправлено действие (dispatch action).

В этом руководстве мы будем строить асинхронное приложение, которое будет использовать API Reddit'а для отображения текущих заголовков для выбранного subreddit'a. Так как же асинхронность вписывается в Redux поток (flow)?

## Действия (Actions)

Когда Вы вызываете асинхронное API, есть два ключевых момента времени: момент непосредственного вызова и момент получения ответа или timeout'a.

Для каждого из этих моментов обычно может требоваться изменение состояния приложения (application state). Для изменения состояния Вы должны запустить (dispatch) нормальное действие, которое будет обработано редьюсером синхронно. Обычно для любого API запроса Вам понадобится запустить (dispatch) по крайней мере три разных вида действий (actions):

* **Действие, информирующее редьюсер о том, что запрос начался.**

  Редьюсер может обрабатывать такой вид действия, переключая флаг `isFetching` в состоянии приложения. Именно так UI понимает, что самое время показать лоадер/спиннер.

* **Действие, информирующее редьюсер о том, что запрос успешно завершился.**

  Редьюсер может обрабатывать такой вид действия, объединяя полученные из запроса данные с тем срезом состояния, которым управляет этот редьюсер и сбрасывая флаг `isFetching`.

* **Действие, информирующее редьюсер о том, что запрос завершился неудачей.**

  Редьюсер может обрабатывать такой вид действия сбрасывая флаг `isFetching`. Также некоторые редьюсеры могут хотеть сохранить сообщение об ошибке, чтобы UI мог его отобразить.

Для всего этого Вы можете использовать поле `status` в действиях (actions):

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

Или Вы можете обявить отдельные типы для таких действий:

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

Выбор использования одного типа действий с флагами или нескольких отдельных типов действий остается за Вами. Это соглашение, которое Вы должны утвердить с Вашей командой. 

Использование нескольких типов действий оставляют меньше места для ошибки, но это не проблема, если Вы генерируете действия и редьюсеры с помощью таких библиотек, как [redux-actions](https://github.com/acdlite/redux-actions).

Какое бы соглашение Вы не выбрали, следуйте ему на протяжении всего приложения.
В этом руководстве мы будем использовать несколько отдельных типов действий.

## Синхронные генераторы действий (Synchronous Action Creators).

Давайте начнем с объявления нескольких синхронный типов и генераторов действий, которые нам понадобятся в нашем приложении. Тут пользователь может выбрать reddit для отображения:

#### `actions.js`

```js
//тип действия
export const SELECT_REDDIT = 'SELECT_REDDIT';

//генератор действия
export function selectReddit(reddit) {
  return {
    type: SELECT_REDDIT,
    reddit
  };
}
```

Также пользователь может нажать кнопку "обновить" для обновления:

```js
export const INVALIDATE_REDDIT = 'INVALIDATE_REDDIT';

export function invalidateReddit(reddit) {
  return {
    type: INVALIDATE_REDDIT,
    reddit
  };
}
```

Это были действия, отвечающие за взаимодействие с пользователем. Также у нас будет и другой тип действий, отвечающий за сетевые запросы. Сейчас мы просто определим их, а чуть позже, посмотрим, как посылать такие действия.

Когда нужно будет фетчить посты для какого-нибудь reddit'a мы будем посылать действие `REQUEST_POSTS`:

```js
export const REQUEST_POSTS = 'REQUEST_POSTS';

export function requestPosts(reddit) {
  return {
    type: REQUEST_POSTS,
    reddit
  };
}
```

Важно чтобы это действие было отделено от `SELECT_REDDIT` и `INVALIDATE_REDDIT`. Хотя они, действия, могут происходить одно за другим, но с возрастанием сложности приложения, Вам может понадобиться фетчить какие-то данные независимо от действий пользователя, например, для предзагрузки самых популярных reddit'ов или для того, чтобы изредка обновлять устаревшие данные. Вы можете также захотеть фетчить данные в ответ на смену роута, т.ч. не очень мудро связывать обновление данных с каким-то определенным UI-событием.

Наконец, когда сетевой запрос будет осуществлен, мы отправим действие `RECEIVE_POSTS`:

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS';

export function receivePosts(reddit, json) {
  return {
    type: RECEIVE_POSTS,
    reddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  };
}
```

Это все, что нам нужно знать на текущий момент. Конкретный механизм работы этих действий вместе с сетевыми запросами будет обсуждаться позже.

>##### Обратите внимание на обработчик ошибки (Note on Error Handling)

> В реальном приложении, Вам бы также понадобилось отправлять действие в случае завершения сетевого запроса ошибкой. В этом руководстве мы не будем реализовывать обработку ошибок, но, [пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует один из возможных путей ее реализации.

## Разработка структуры хранилища (Designing the State Shape)

Как и в базовом руководстве, Вам нужно [разработать структуру состояния приложения](../basics/Reducers.md#designing-the-state-shape), прежде чем начинать писать само приложение. В случае с асинхронным кодом, появляется больше состояний о которых нужно позаботиться, т.ч. нам нужно все как следует обдумать.

Часто именно эта часть сбивает с толку новичков, потому что сразу не ясно, какая информация описывает состояние в асинхронном приложении и как организовать все это, в одно дерево состояния.

Мы начнем с наиболее общего варианта использования - списков. Веб-приложения часто отображают списки чего-либо. Например, список постов или список друзей. Вам нужно будет решить какие типы списков сможет отображать ваше приложение. Вам нужно хранить их отдельно в состоянии, потому что в этом случае Вы можете кешировать их и снова обновлять данные по необходимости. 

Вот как может выглядеть структура состояния для нашего  приложения “Reddit headlines”:

```js
{
  selectedReddit: 'frontend',
  postsByReddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [{
        id: 42,
        title: 'Confusion about Flux and Relay'
      }, {
        id: 500,
        title: 'Creating a Simple Application Using React JS and Flux Architecture'
      }]
    }
  }
}
```

тут есть несколько важных моментов:

* Мы храним информацию о каждом subreddit’е отдельно, следовательно мы можем кешировать любой subreddit. Когда пользователь переключается между ними во второй раз, обновление UI будет мгновенным и сможем не перезагружать данные если мы этого не хотим. Не переживайте о том, что все эти элементы (subreddit'ы, а их может быть очень много) будут находиться в памяти: Вам не понадобятся никакие чистки памяти, если только Вы и Ваш пользователь не имеете дело с десятками тысяч элементов и при этом пользователь очень редко закрывает вкладку браузера.

* Для каждого списка элементов Вы захотите хранить `isFetching` для показа спиннера, `didInvalidate`, который Вы потом сможете установить в true, если данные устрареют, `lastUpdated` для того, тчобы знать когда данные были обновлены в последний раз и собственно `items`. В реальном приложении Вы также захотите хранить состояние страничной навигации: `fetchedPageCount` и `nextPageUrl`.

>##### Обратите внимание на вложенные сущности

> В этом примере мы храним полученные элементы вместе с информацией о постраничной навигации. Однако этот подход будет очень плох, если у Вас будут вложенные сущности, которые ссылаются друг на друга или если Вы дадите пользователю возможность редактировать элементы. Представьте, что пользователь хочет отредактировать загруженный пост, но этот пост сдублирован в нескольких местах в дереве состояния (state tree). Реализация такого будет очень болезненна.

>Если у Вас есть вложенные сущности или если Вы даете возможность редактировать загруженные элементы, то Вы должны хранить их отдельно в состоянии (state), как если бы оно, (состояние), было базой данных. И в информации о постраничной навигации Вы можете ссылаться на такие элементы только по их ID. Это позволит Вам всегда держать их в актуальном состоянии. [Пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует этот подход вместе с [normalizr](https://github.com/gaearon/normalizr) для упорядочивания вложенных API ответов. С таким подходом Ваше состояние (state) может выглядеть вот так:

>```js
> {
>   selectedReddit: 'frontend',
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
>   postsByReddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [42, 100]
>     }
>   }
> }
>```

>В этом руководстве мы не будем упорядочивать сущности, но это то, о чем Вам стоит задуматься для более динамичного приложения.

## Обработка действий (Handling Actions)

Перед тем, как переходить к деталям отправки действий (dispatching actions) вместе с сетевми запросами, мы напишем редьюсеры для действий, которые определили выше.

>##### Обратите внимание на компановку редьюсеров (Reducer Composition)

> Предполагается, что Вы понимаете что такое компоновка редьюсеров с помощью функции [`combineReducers()`](../api/combineReducers.md), описанной в разделе [Разделение редьюсеров](../basics/Reducers.md#splitting-reducers) в [базовом руководстве](../basics/README.md). Если это не так, то пожалуйста [сначала прочтите это](../basics/Reducers.md#splitting-reducers).

#### `reducers.js`

```js
import { combineReducers } from 'redux';
import {
  SELECT_REDDIT, INVALIDATE_REDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions';

function selectedReddit(state = 'reactjs', action) {
  switch (action.type) {
  case SELECT_REDDIT:
    return action.reddit;
  default:
    return state;
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
  case INVALIDATE_REDDIT:
    return Object.assign({}, state, {
      didInvalidate: true
    });
  case REQUEST_POSTS:
    return Object.assign({}, state, {
      isFetching: true,
      didInvalidate: false
    });
  case RECEIVE_POSTS:
    return Object.assign({}, state, {
      isFetching: false,
      didInvalidate: false,
      items: action.posts,
      lastUpdated: action.receivedAt
    });
  default:
    return state;
  }
}

function postsByReddit(state = {}, action) {
  switch (action.type) {
  case INVALIDATE_REDDIT:
  case RECEIVE_POSTS:
  case REQUEST_POSTS:
    return Object.assign({}, state, {
      [action.reddit]: posts(state[action.reddit], action)
    });
  default:
    return state;
  }
}

const rootReducer = combineReducers({
  postsByReddit,
  selectedReddit
});

export default rootReducer;
```

Две части этого кода вызывают особый интерес:

* Мы используем ES6-синтаксис вычисляемого свойства, т.е. мы можем обновить `state[action.reddit]` с помощью `Object.assign()` с использованием меньшего количества строк кода. Вот это:

  ```js
  return Object.assign({}, state, {
    [action.reddit]: posts(state[action.reddit], action)
  });
  ```
  эквивалентно этому:

  ```js
  let nextState = {};
  nextState[action.reddit] = posts(state[action.reddit], action);
  return Object.assign({}, state, nextState);
  ```
* Мы извлекли `posts(state, action)`, который управляет состоянием конкретного списка постов. Это просто [компоновка редьюсеров](../basics/Reducers.md#splitting-reducers)! Нам выбирать, как разбить/разделить редьюсер на более мелкие редьюсеры и в этом случае, мы доверяем обновление элементов внутри объекта функции-редьюсеру `posts`. [Пример из реальной жизни](../introduction/Examples.html#real-world) идет еще дальше, показывая, как создавать фабрику редьюсеров для параметризированных редьюсеров постраничной навигации.

Помните, что редьюсеры - это всего лишь функции, т.е. вы можете использовать функциональную композицию (речь о функциональном подходе к программированию и композиции функций *прим. переводчика*) и функции высшего порядка так часто, как Вам это будет удобно.

## Асинхронные генераторы действий (Async Action Creators)

Наконец, как мы используем синхронные генераторы действий, [созданные нами ранее](#synchronous-action-creators) вместе с сетевыми запросами? Стандартый для Redux путь - это использование [Redux Thunk middleware](https://github.com/gaearon/redux-thunk). Этот посредник (middleware) содержится в отдельном пакете, который называется `redux-thunk`. Мы поясним, как работают посредники [позже](Middleware.md). Сейчас есть одна важная вещь, о которой вам нужно знать: при использовании конкретно этого посредника, генератор действий может вернуть функцию, вместо объекта действия. Таким образом, генератор действия превращается в [преобразователь (Thunk)](https://en.wikipedia.org/wiki/Thunk)

Когда генератор действия вернет функцию, эта функция будет вызвана посредником Redux Thunk. Этой функции не обязательно быть чистой. Таким образом, в ней разрешается инициировать побочные эффекты (side effects), в том числе и асинхронные вызовы API. Также, эти функции могут вызывать действия (dispatch actions), такие же синхронные действия, которые мы отправляли ранее.

Мы все еще можем определить эти специальные thunk-генераторы действий внутри нашего `actions.js` файла:

#### `actions.js`

```js
import fetch from 'isomorphic-fetch';

export const REQUEST_POSTS = 'REQUEST_POSTS';
function requestPosts(reddit) {
  return {
    type: REQUEST_POSTS,
    reddit
  };
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(reddit, json) {
  return {
    type: RECEIVE_POSTS,
    reddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  };
}

// Тут мы встречаемся с нашим первым thunk-генератором действий!
// Хотя его содержимое отличается, вы должны использовать его, как и любой другой генератор действий:
// store.dispatch(fetchPosts('reactjs'));

export function fetchPosts(reddit) {

  // Thunk middleware знает, как обращаться с функциями.
  // Он передает метод действия в качестве аргумента функции,
  // т.о, это позволяет отправить действие самостоятельно.

  return function (dispatch) {

    // First dispatch: the app state is updated to inform
    // that the API call is starting.

    dispatch(requestPosts(reddit));

    // The function called by the thunk middleware can return a value,
    // that is passed on as the return value of the dispatch method.

    // In this case, we return a promise to wait for.
    // This is not required by thunk middleware, but it is convenient for us.

    return fetch(`http://www.reddit.com/r/${reddit}.json`)
      .then(response => response.json())
      .then(json =>

        // We can dispatch many times!
        // Here, we update the app state with the results of the API call.

        dispatch(receivePosts(reddit, json))
      );

      // In a real world app, you also want to
      // catch any error in the network call.
  };
}
```

>##### Примечание по `fetch`

>Мы используем [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) в примерах. Это новое API для создания сетевых запросов, которое заменяет `XMLHttpRequest` в большинства стандарных случаев. Поскольку большинство браузеров до сих пор не поддеживают его нативно, мы полагаем, что вы для этого используете[`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) библиотеку:

>```js
// Добавьте это в каждый файл, где вы используете `fetch`
>import fetch from 'isomorphic-fetch';
>```

>Внутри она использует [`whatwg-fetch` полифил](https://github.com/github/fetch) на клиенте и [`node-fetch`](https://github.com/bitinn/node-fetch) на сервере, поэтому вам не понадобится менять вызовы API, если вы захотите сделать ваше приложение [универсальным](https://medium.com/@mjackson/universal-javascript-4761051b7ae9).

>Помните, что любой `fetch` полифил предполагает, что [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) полифил уже присутствует. Самый простой способ убедиться, что вы подключили Promise-полифил - это подключить ES6-полифил Babel во входной точке, прежде чем любой другой код запустится:

>```js
>// Добавьте это в самом начале вашего приложения
>import 'babel-core/polyfill';
>```

Как мы добавляем посредника Redux Thunk в механизм диспетчера? Для этого мы используем метод [`applyMiddleware()`](../api/applyMiddleware.md) из Redux, как показано ниже:

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk';
import createLogger from 'redux-logger';
import { createStore, applyMiddleware } from 'redux';
import { selectReddit, fetchPosts } from './actions';
import rootReducer from './reducers';

const loggerMiddleware = createLogger();

const createStoreWithMiddleware = applyMiddleware(
  thunkMiddleware, // lets us dispatch() functions
  loggerMiddleware // neat middleware that logs actions
)(createStore);

const store = createStoreWithMiddleware(rootReducer);

store.dispatch(selectReddit('reactjs'));
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
);
```

Хорошая новость о преобразователях - они могут направлять результаты друг другу:

#### `actions.js`

```js
import fetch from 'isomorphic-fetch';

export const REQUEST_POSTS = 'REQUEST_POSTS';
function requestPosts(reddit) {
  return {
    type: REQUEST_POSTS,
    reddit
  };
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(reddit, json) {
  return {
    type: RECEIVE_POSTS,
    reddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  };
}

function fetchPosts(reddit) {
  return dispatch => {
    dispatch(requestPosts(reddit));
    return fetch(`http://www.reddit.com/r/${reddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(reddit, json)));
  };
}

function shouldFetchPosts(state, reddit) {
  const posts = state.postsByReddit[reddit];
  if (!posts) {
    return true;
  } else if (posts.isFetching) {
    return false;
  } else {
    return posts.didInvalidate;
  }
}

export function fetchPostsIfNeeded(reddit) {

  // Note that the function also receives getState()
  // which lets you choose what to dispatch next.

  // This is useful for avoiding a network request if
  // a cached value is already available.

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), reddit)) {
      // Dispatch a thunk from thunk!
      return dispatch(fetchPosts(reddit));
    } else {
      // Let the calling code know there's nothing to wait for.
      return Promise.resolve();
    }
  };
}
```

Это позволяет нам писать более сложный поток асинхронного управления постепенно, в то время, как потребляющий код может оставаться таким же, довольно долгое время:

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState());
);
```

>##### Примечание о серверном рендеринге

> Асинхронные генераторы действий особенно удобны для серверного рендеринга. Вы можете создать хранилище, вызвать отдельный асинхронный генератор действия, который вызовет другие асинхронные генераторы действия для выборки данных для всей части вашего приложения и отрендерит, только после того, как promise его вернет. Затем ваше хранилище будет полностью гидратировано с состоянием необходимым для рендеринга.

[Thunk middleware](https://github.com/gaearon/redux-thunk) - это не единственный путь управления асинхронными действиями в Redux. Вы можете использовать [redux-promise](https://github.com/acdlite/redux-promise) или [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) для отправки Promises вместо функций. Вы можете отправлять Наблюдателей (Observables) с [redux-rx](https://github.com/acdlite/redux-rx). Вы даже можете писать собственных посредников (middleware), для описания вызовов вашего API, как например в [real world example](../introduction/Examples.html#real-world). Решать вам, попробовать несколько вариантов, выбрать конвенции которые вам нравятся и следовать им, будь то с использованием посредника или без него.

## Подключение к UI

Отправка асинхронных действий не отличается от отправки синхронных действий, поэтому мы не будем обсуждать это в деталях. Почитайте [использование в связке с React](../basics/UsageWithReact.md) для знакомства с использованием Redux с React-компонентами. С полным исходным кодом, обсуждаемым в этом примере, вы можете ознакомится в [примере: Reddit API](ExampleRedditAPI.md)

## Следующие шаги

Прочтите [Асинхронный поток](AsyncFlow.md) для резюмирования того, как асинхронные действия вписываются в поток Redux.
