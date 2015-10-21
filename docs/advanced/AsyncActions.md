# Асинхронные действия (Async Actions)

В [базовом руководстве](../basics/README.md) мы построили простое приложение - список дел. Оно было совершенно синхронным. Состояние (State) обновлялось сразуже после того, как было сгенерировано и отправлено действие (dispatch action).

В этом руководстве мы будем строить асинхронное приложение, которое будет использовать API Reddit'а для отображения текущих заголовков для выбранного subreddit'a. Так как же асинхронность вписывается в Redux поток (flow)?

## Действия (Actions)

Когда Вы вызываете асинхронное API есть два ключевых момента времени: момент непосредственного вызова и момент получения ответа (или timeout'a)

Для каждого из этих моментов обычно может требоваться изменение состояния приложения (application state). Для изменения состояния Вы должны запустить (dispatch) нормальное действие, которое будет обработано редьюсером синхронно. Обычно для любого API запроса Вам понадобится запустить (dispatch) по крайней мере три разных вида действий (actions):

* **Действие, информирующее редьюсер о том, что запрос начался.**

  Редьюсер может обрабатывать такой вид действия, переключая флаг `isFetching` в состоянии приложения. Именно так UI понимает, что самое время показать лоадер/спиннер.

* **Действие, информирующее редьюсер о том, что запрос успешно завершился.**

  Редьюсер может обрабатывать такой вид действия, объединяя полученные из запроса данные с тем срезом состояния, которым управляет этот редьюсер, и сбрасывая флаг `isFetching`.

* **Действие, информирующее редьюсер о том, что запрос завершился неудачей.**

  Редьюсер может обрабатывать такой вид действия сбрасывая флаг `isFetching`. Также некоторые редьюсеры могут хотеть сохранить сообщение об ошибке, чтобы UI мог его отобразить.

Для всего этого Вы можете использовать поле `status` в действиях (actions):

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

Или Вы можете обхявить отдельные типы для таких действий:

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

Также пользователь может нажать кнопку "обновтить" для обновления:

```js
export const INVALIDATE_REDDIT = 'INVALIDATE_REDDIT';

export function invalidateReddit(reddit) {
  return {
    type: INVALIDATE_REDDIT,
    reddit
  };
}
```

Это были действия, отвечающие за взаимодействие с пользователем. Также у нас будет и другой тип действий, отвечающий за сетевые запросы. Сейчас мы просто определим их, а чуть пзже посмотрим как посылать такие действия.

Когда нужно будет фетчить посты для какого-нибудь reddit'a мы будем посылать дейсвие `REQUEST_POSTS`:

```js
export const REQUEST_POSTS = 'REQUEST_POSTS';

export function requestPosts(reddit) {
  return {
    type: REQUEST_POSTS,
    reddit
  };
}
```

Важно, чтобы это действие было отделено от `SELECT_REDDIT` и `INVALIDATE_REDDIT`. Хотя они (действия) могут происходить одно за другим, но, с возрастанием сложности приложения, Вам может понадобиться фетчить какие то данные независимо от действий пользователя (например для предзагрузки самых популярных reddit'ов или для того, чтобы изредка обновлять устаревшие данные). Вы можете  также захотеть фетчить данные в ответ на смену роута, так что не очень мудро связывать обновление данных с каким то обпределенным UI событием.

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

> В реальном приложении, Вам бы также понадобилось отправлять действие в случае завершения сетевого запроса ошибкой. В этом руководстве мы не будем реализовывать обработку ошибок, но [пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует один из возможных путей ее реализации.

## Разработка структуры хранилища (Designing the State Shape)

Как и в базовом руководстве, Вам нужно [разработать структуру состояния приложения](../basics/Reducers.md#designing-the-state-shape) прежде чем начинать писать само приложение. В случае с асинхронным кодом появляется больше состояний, о которых нужно позаботиться, так что нам нужно все как следует обдумать.

Часто именно эта часть сбивает с толку новичков потому, что сразу не ясно какая информация описывает состояние в асинхронном приложении и как организовать все это в одно дерево состояния.

Мы начнем с наиболее общего варианта использования - списокв. Веб приложения часто отображают списки чего-либо. Например список постов или список друзей. Вам нужно будет решить какие типы списков сможет отображать ваше приложение. Вам нужно хранить их отдельно в состоянии потому, что в этом случае Вы можете кешировать их и снова обновлять данные по необходимости. 

We’ll start with the most common use case: lists. Web applications often show lists of things. For example, a list of posts, or a list of friends. You’ll need to figure out what sorts of lists your app can show. You want to store them separately in the state, because this way you can cache them and only fetch again if necessary.

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

* Мы храним информацию о каждом subreddit’е отдельно, следовательно мы можем кешировать любой subreddit. Когда пользователь переключается между ними во торой раз, обновление UI будет мгновенным, и сможем не перезагружать данные если мы этого не хотим. Не переживайт ео том, тчо все эти желементы (subreddit'ы, а их может быть очень много) будут находиться в памяти: Вам не понадобятся никакие читски памяти если только Вы и Ваш пользователь не имеете дело с десятками тысяч элементов и приэтом пользователь очень редко закрывает вкладку браузера.

* Для каждого списка элементов Вы захотите хранить `isFetching` для показа спиннера, `didInvalidate`, который Вы потом сможете установить в true, если данные устраеют, `lastUpdated` для того, тчобы знать когда данные были обновлены в последний раз и собственно `items`. В реально приложении Вы также захотите хранить состояние страничной навигации: `fetchedPageCount` и `nextPageUrl`.

>##### Обратите внимание на вложенные сущности

> В этом примере мы храним полученные элементы вместе с информацией о постраничной навигации. Однако этот подход будет очень плох, если у Вас будут вложенные сущности, которые ссылаются друг на друга, или если Вы дадите пользователю возможность редактировать элементы. Представьте, что пользователь хочет отредактировать загруженный пост, но этот пост сдублирован в нескольких местах в дереве состояния (state tree). Реализация такого будет очень болезненна.

>Если у Вас есть вложенные сущности или если Вы даете возможность редактировать загруженные элементы, то Вы должны хранить их отдельно в состоянии (state), как есди бы оно (состояние) было базой данных. И в информации о постраничной навигации Вы можете ссылаться на такие элементы только по их ID. Это позволит Вам всегда держать их в актуальном состоянии. [Пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует это подход вместе с [normalizr](https://github.com/gaearon/normalizr) для упорядочивания вложенных API ответов. С таким подходом Ваше состояние (state) может выглядеть вот так:

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

> Прдполагается, что Вы понимаете что такое компановка редьюсеров с помощью функции [`combineReducers()`](../api/combineReducers.md) описанной в разделе [Разделение редьюсеров](../basics/Reducers.md#splitting-reducers) в [базовом руководстве](../basics/README.md). Если это не так, то пожалуйста [сначала прочтите это](../basics/Reducers.md#splitting-reducers).

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

* Мы испозьзуем ES6 синтаксис вычисляемого свойства, т.е. мы можем обновить `state[action.reddit]` с помощью `Object.assign()` с использованием меньшего количества строк кода. Вот это:

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
* Мы извлекли `posts(state, action)`, который управляет состоянием конкретного списка постов. Это просто [компановка редьюсеров](../basics/Reducers.md#splitting-reducers)! Нам выбирать как разбить/разделить редьюсер на более мелкие редьюсеры и, в этом случае, мы доверяем обновление элементов внутри объекта функции-редьюсеру `posts`. [Пример из реальной жизни](../introduction/Examples.html#real-world) идет еще дальше, показывая как фабрику редьюсеров для параметризированных редьюсеров постраничной навигации.

Помние, что редьюсеры - это всего лишь функции, т.е. вы можете использовать функцилнальную композицию (речь о функциональном подходе к программированию и композиции функций *прим. переводчика*) и функции высшего порядка так часто, как Вам это будет удобно.

## Асинхронные генераторы действий (Async Action Creators)

Наконец как мы используем синхронные генераторы действий, [созданные нами ранее](#synchronous-action-creators) вместе с сетевыми запросами? Стандартый для Redux путь - это испольщование [Redux Thunk middleware](https://github.com/gaearon/redux-thunk). Этот миддвар (middleware) содержитмя в отдельном пакете, который называется `redux-thunk`. Мы поясним как работают миддлвары [позже](Middleware.md). Сейчас есть одна важная вещь, о котрой вам нужно знать: при использовании конкретно этого миддлвара генератор действий может вернуть функцию вместо объекта действия. Таким образом, генератор действия превращается в [преобразователь (Thunk)](https://en.wikipedia.org/wiki/Thunk)

Когда генератор действия вернет функцию, эта функция бкдет вызвана Redux Thunk миддлваром. Этой функции не обязательно быть чистой. Таким образом, в ней разрешается инициировать побочные эффекты (side effects), в том числе и асинхронные вызовы API. Также эти функции могут отправлять действия (dispatch actions), такие же синхронные действия, которые мы отправляли ранее.

Мы все еще можем определить эти специальные thunk-генераторы-действий внутри нашего `actions.js` файла:

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

// Тут мы встречаемся с нашим первым thunk-генератором-действий!
// Though its insides are different, you would use it just like any other action creator:
// store.dispatch(fetchPosts('reactjs'));

export function fetchPosts(reddit) {

  // Thunk middleware knows how to handle functions.
  // It passes the dispatch method as an argument to the function,
  // thus making it able to dispatch actions itself.

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

>##### Note on `fetch`

>We use [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) in the examples. It is a new API for making network requests that replaces `XMLHttpRequest` for most common needs. Because most browsers don’t yet support it natively, we suggest that you use [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) library:

>```js
// Do this in every file where you use `fetch`
>import fetch from 'isomorphic-fetch';
>```

>Internally, it uses [`whatwg-fetch` polyfill](https://github.com/github/fetch) on the client, and [`node-fetch`](https://github.com/bitinn/node-fetch) on the server, so you won’t need to change API calls if you change your app to be [universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9).

>Be aware that any `fetch` polyfill assumes a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) polyfill is already present. The easiest way to ensure you have a Promise polyfill is to enable Babel’s ES6 polyfill in your entry point before any other code runs:

>```js
>// Do this once before any other code in your app
>import 'babel-core/polyfill';
>```

How do we include the Redux Thunk middleware in the dispatch mechanism? We use the [`applyMiddleware()`](../api/applyMiddleware.md) method from Redux, as shown below:

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

The nice thing about thunks is that they can dispatch results of each other:

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

This lets us write more sophisticated async control flow gradually, while the consuming code can stay pretty much the same:

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState());
);
```

>##### Note about Server Rendering

>Async action creators are especially convenient for server rendering. You can create a store, dispatch a single async action creator that dispatches other async action creators to fetch data for a whole section of your app, and only render after the Promise it returns, completes. Then your store will already be hydrated with the state you need before rendering.

[Thunk middleware](https://github.com/gaearon/redux-thunk) isn’t the only way to orchestrate asynchronous actions in Redux. You can use [redux-promise](https://github.com/acdlite/redux-promise) or [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) to dispatch Promises instead of functions. You can dispatch Observables with [redux-rx](https://github.com/acdlite/redux-rx). You can even write a custom middleware to describe calls to your API, like the [real world example](../introduction/Examples.html#real-world) does. It is up to you to try a few options, choose a convention you like, and follow it, whether with, or without the middleware.

## Connecting to UI

Dispatching async actions is no different from dispatching synchronous actions, so we won’t discuss this in detail. See [Usage with React](../basics/UsageWithReact.md) for an introduction into using Redux from React components. See [Example: Reddit API](ExampleRedditAPI.md) for the complete source code discussed in this example.

## Next Steps

Read [Async Flow](AsyncFlow.md) to recap how async actions fit into the Redux flow.
