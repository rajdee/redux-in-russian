# Асинхронные действия (Async Actions)

В [базовом руководстве](../basics/README.md) мы построили простое приложение - список дел. Оно было совершенно синхронным. Состояние (State) обновлялось сразуже после того, как было сгенерировано и отправлено действие (dispatch action).

В этом руководстве мы будем строить асинхронное приложение, которое будет использовать API Reddit'а для отображения текущих заголовков для выбранного subreddit'a. Так как же асинхронность вписывается в Redux поток (flow)?

## Действия (Actions)

Когда Вы обращаетесь к асинхронному API есть два ключевых момента времени: момент непосредственного обращения и момент получения ответа (или timeout'a)

Обычно для каждого из этих моментов может требоваться изменение состояния приложения (application state). Для изменения состояния Вы должны запустить (dispatch) действие, которое будет обработано редьюсером синхронно. Обычно для любого API запроса Вам понадобится запустить (dispatch) по крайней мере три разных вида действий (actions):

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

Или можно объявить отдельные типы для таких действий:

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

Выбор использования одного типа действий с флагами или нескольких отдельных типов действий остается за Вами. Это соглашение, которое Вы должны утвердить с Вашей командой. 

Использование нескольких типов действий оставляет меньше места для ошибки, но это не проблема, если Вы генерируете действия и редьюсеры с помощью таких библиотек, как [redux-actions](https://github.com/acdlite/redux-actions).

Какое бы соглашение Вы не выбрали, следуйте ему на протяжении всего приложения.
В этом руководстве мы будем использовать несколько отдельных типов действий.

## Синхронные генераторы действий (Synchronous Action Creators).

Давайте начнем с объявления нескольких синхронных типов действий и их генераторов, которые нам понадобятся в приложении. Тут пользователь может выбрать reddit для отображения:

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

Также пользователь может нажать кнопку "обновить" для обновления reddit'a:

```js
export const INVALIDATE_REDDIT = 'INVALIDATE_REDDIT';

export function invalidateReddit(reddit) {
  return {
    type: INVALIDATE_REDDIT,
    reddit
  };
}
```

Это были действия, отвечающие за взаимодействие с пользователем. Также у нас будет и другой тип действий, отвечающий за сетевые запросы. Сейчас мы просто определим их, а чуть позже посмотрим как посылать такие действия.

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

Важно чтобы это действие было отделено от `SELECT_REDDIT` и `INVALIDATE_REDDIT`. Хотя они (действия) и могут происходить одно за другим, но, с возрастанием сложности приложения, Вам может понадобиться фетчить какие то данные независимо от действий пользователя (например для предзагрузки самых популярных reddit'ов или для того, чтобы изредка обновлять устаревшие данные). Вы можете также захотеть фетчить данные в ответ на смену роута, так что не очень мудро связывать обновление данных с каким то определенным UI событием.

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

> В реальном приложении Вам бы также понадобилось отправлять действие (dispatch action) в случае завершения сетевого запроса ошибкой. В этом руководстве мы не будем реализовывать обработку ошибок, но [пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует один из возможных путей ее реализации.

## Проектирование структуры хранилища (Designing the State Shape)

Как и в базовом руководстве, Вам нужно [спроектировать структуру состояния приложения](../basics/Reducers.md#designing-the-state-shape) прежде чем начинать писать само приложение. В случае с асинхронным кодом появляется больше состояний, о которых нужно позаботиться, так что нам нужно все как следует обдумать.

Часто именно эта часть сбивает с толку новичков потому, что сразу не ясно какая информация описывает состояние в асинхронном приложении и как организовать все это в одно дерево состояния.

Мы начнем с наиболее общего варианта использования - списокв. Веб приложения часто отображают списки чего-либо. Например список постов или список друзей. Вам нужно будет решить какие типы списков сможет отображать ваше приложение. Вам нужно хранить их отдельно в состоянии потому, что в этом случае Вы можете кешировать их и снова обновлять данные по необходимости. 

Вот как может выглядеть структура состояния для нашего приложения “Reddit headlines”:

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

* Мы храним информацию о каждом subreddit’е отдельно, следовательно, мы можем кешировать любой subreddit. Когда пользователь переключается между ними во торой раз, обновление UI будет мгновенным, и мы сможем не перезагружать данные если мы этого не хотим. Не переживайте о том, что все эти элементы (subreddit'ы, а их может быть очень много) будут находиться в памяти: Вам не понадобятся никакие читски памяти если только Вы и Ваш пользователь не имеете дела с десятками тысяч элементов и приэтом пользователь очень редко закрывает вкладку браузера.

* Для каждого списка элементов Вы захотите хранить `isFetching` для показа спиннера, `didInvalidate`, который Вы потом сможете установить в true, если данные устраеют, `lastUpdated` для того, чтобы знать когда данные были обновлены в последний раз и собственно `items`. В реальном приложении Вы также захотите хранить состояние страничной навигации: `fetchedPageCount` и `nextPageUrl`.

>##### Обратите внимание на вложенные сущности

> В этом примере мы храним полученные элементы вместе с информацией о постраничной навигации. Однако, этот подход будет очень плох, если у Вас будут вложенные сущности, которые ссылаются друг на друга, или если Вы дадите пользователю возможность редактировать элементы. Представьте, что пользователь хочет отредактировать загруженный пост, но этот пост сдублирован в нескольких местах в дереве состояния (state tree). Реализация такого редактирования будет очень болезненна.

>Если у Вас есть вложенные сущности или если Вы даете возможность редактировать загруженные элементы, то Вы должны хранить их отдельно в состоянии (state), как если бы оно (состояние) было базой данных. И в информации о постраничной навигации Вы можете ссылаться на такие элементы только по их ID. Это позволит Вам всегда держать их в актуальном состоянии. [Пример из реальной жизни](../introduction/Examples.html#real-world) демонстрирует этот подход вместе с [normalizr](https://github.com/gaearon/normalizr) для упорядочивания вложенных API ответов. С таким подходом Ваше состояние (state) может выглядеть вот так:

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
* Мы извлекли `posts(state, action)`, который управляет состоянием конкретного списка постов. Это просто [компановка редьюсеров](../basics/Reducers.md#splitting-reducers)! Нам выбирать как разбить/разделить редьюсер на более мелкие редьюсеры и, в этом случае, мы доверяем обновление элементов внутри объекта функции-редьюсеру `posts`. [Пример из реальной жизни](../introduction/Examples.html#real-world) идет еще дальше, показывая как создать фабрику редьюсеров для параметризированных редьюсеров постраничной навигации.

Помние, что редьюсеры - это всего лишь функции, т.е. вы можете использовать функцилнальную композицию (речь о функциональном подходе к программированию и композиции функций *прим. переводчика*) и функции высшего порядка так часто, как Вам это будет удобно.

## Асинхронные генераторы действий (Async Action Creators)

Так как же мы используем синхронные генераторы действий, [созданные нами ранее](#synchronous-action-creators) вместе с сетевыми запросами? Стандартый для Redux путь - это использование [Redux Thunk middleware](https://github.com/gaearon/redux-thunk). Этот миддлвар (middleware) содержится в отдельном пакете, который называется `redux-thunk`. Мы поясним как работают миддлвары [позже](Middleware.md). Сейчас есть одна важная вещь, о котрой Вам нужно знать: при использовании конкретно этого миддлвара генератор действий может вернуть функцию вместо объекта действия. Таким образом, генератор действия превращается в [преобразователь (Thunk)](https://en.wikipedia.org/wiki/Thunk)

Когда генератор действий вернет функцию, эта функция будет вызвана Redux Thunk миддлваром. Этой функции не обязательно быть чистой. Таким образом, в ней разрешается инициировать побочные эффекты (side effects), в том числе и асинхронные вызовы API. Также эти функции могут отправлять действия (dispatch actions), точно такие же синхронные действия, которые мы отправляли ранее.

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
// Не смотря на то, что его внутренности немного отличаются от того, что мы видели ранее, Вы все равно будете использовать его как любой другой генератор действий:
// store.dispatch(fetchPosts('reactjs'));

export function fetchPosts(reddit) {

  // Thunk-middleware знает что дальше делать с функцией, которую возвращает редьюсер fetchPosts.
  // Он передает ей метод dispatch как аргумент,
  // что делает ее (эту функцию) способной самостоятельно отправить (dispatch) действие.

  return function (dispatch) {

    // Первая отправка действия (dispatch): обновляется состояние приложения (state). 
    // Таким образом, мы объявляем, что начинается вызов API.

    dispatch(requestPosts(reddit));

    // Функция, вызываемая thunk-миддлваром, может возвращать значение,
    // которое передается, как возвращаемое значение в метод dispatch,
    // в который редьюсер fetchPosts был передан как аргумент 
    // (store.dispatch(fetchPosts('reactjs'));).

    // В данном случае мы возвращаем promise.
    // Это не обязательно для thunk-миддлвара, но это удобно для нас.

    return fetch(`http://www.reddit.com/r/${reddit}.json`)
      .then(response => response.json())
      .then(json =>

        // Мы можем отправлять событие (dispatch) много раз!
        // Здесь мы обновляем состояние приложения (state) результатами вызова API.

        dispatch(receivePosts(reddit, json))
      );

      // В реально приложении Вы также захотите ловить 
      // и обрабатывать любые сетевые ошибки.
  };
}
```

>##### Обратите внимание на `fetch`

>В это м примере Мы используем [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API). Это новый API для осуществления сетевых запросов, который призван заменить `XMLHttpRequest` в большинстве случаев. Поскольку пока еще не все браузеры поддерживают его нативно, мы предполашаем, что Вы пользуетесь библиотекой [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch):

>```js
// Делайте так в каждом файле, в котором используете `fetch`
>import fetch from 'isomorphic-fetch';
>```

>Внутри, эта библиотека использует [`whatwg-fetch` полифил](https://github.com/github/fetch) на стороне клиента и [`node-fetch`](https://github.com/bitinn/node-fetch) на стороне сервера, т.о. Вам не нужно будет менять вызовы API в случае, если Вы перепишете свое приложение так, что оно станет [универсальным](https://medium.com/@mjackson/universal-javascript-4761051b7ae9).

>Имейте ввиду, что любой `fetch` полифил рассчитывает на присутствие [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) полифила. Простейший путь убедиться в том, что уВас есть Promise полифил - активировать Babel ES6 полифил в точке входа приложения перед тем, как выполнять какой-либо код:

>```js
>// Сделайте это один раз перед любым выполнения любого другого кода в Вашем приложении
>import 'babel-core/polyfill';
>```

Как же мы включаем Redux Thunk миддлвар в механизм отправки действий (dispatch mechanism)? Мы используем [`applyMiddleware()`](../api/applyMiddleware.md) метод из Redux:

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk';
import createLogger from 'redux-logger';
import { createStore, applyMiddleware } from 'redux';
import { selectReddit, fetchPosts } from './actions';
import rootReducer from './reducers';

const loggerMiddleware = createLogger();

const createStoreWithMiddleware = applyMiddleware(
  thunkMiddleware, // позволяет нам отправлять (dispatch()) функции вместо объектов
  loggerMiddleware // аккуратный миддлвар, который логирует действия (actions)
)(createStore);

const store = createStoreWithMiddleware(rootReducer);

store.dispatch(selectReddit('reactjs'));
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
);
```

Приятным моментом при использовании thunks является то, что они могут отправлять (dispatch) результаты друг друга:

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

  // Обратите внимание на то, что функция также принимает getState(),
  // который позвояет Вам выбрать какое действие отправлять (dispatch) следующим.

  // Это полезно для избежания сетевых запросов в том случае,
  // если доступно закешированное значение.

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), reddit)) {
      // Отправка (Dispatch) thunk из thunk!
      return dispatch(fetchPosts(reddit));
    } else {
      // Дать вызывающему коду знать, что ждать больше нечего.
      return Promise.resolve();
    }
  };
}
```

Это позволяет нам писать более сложную асинхронную управляющую логику постепенно, вто время как потребляющий код, который ее использует, может оставаться почти неизменным:

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState());
);
```

>##### Замечание по поводу рендеринга на сервере (Server Rendering)

>Асинхронные генераторы действий особенно удобны для рендеринга на сервере. Вы можете создать хранилище (store), отправить (dispatch) единственный асинхронный генератор действия (async action creator), который отправляет другие асинхронные генераторы действий для выборки данных для всего раздела приложения и отрендерить только после завершения Promise, который он (единственный асинхронный генератор) возвращает. Затем Ваше хранилище (store) уже будет наполнено (hydrated) состоянием, которое Вам нужно перед рендерингом

[Thunk миддлвар](https://github.com/gaearon/redux-thunk) это не единственный способ организовать асинхронные действия (actions) в Redux. Вы можете использовать [redux-promise](https://github.com/acdlite/redux-promise) или [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) для отправки (dispatch) Promis'ов вместо функций. Вы можете отправлять (dispatch) Observables c [redux-rx](https://github.com/acdlite/redux-rx). Вы даже можете писать свои миддлвары для описания вызовов вашего API, как в [примере из реальной жизни](../introduction/Examples.html#real-world). Только от Вас зависит какие арианты попробовать, какой выбрать в качестве наиболее удобного, и использовать его с миддлваром или без.

## Привязка к UI (Connecting to UI)

Отправка (dispatching) асинхронных действий (actions) не отличается от отправки (dispatching) синохронных действий (actions), следовательно мы не будем детально обсуждать это. Взгляните на [Использование с React](../basics/UsageWithReact.md) в качестве введения в использование Redux с React компонентами. Взгляните на [Пример: Reddit API](ExampleRedditAPI.md) для того, чтобы увидеть полный исходный код, который осбуждался в этом примере.

## Следующие шаги

Прочтите [Асинхронный поток (Async Flow)](AsyncFlow.md) для подведения итогов о том, как  асинхронные действия работают в потоке Redux.
