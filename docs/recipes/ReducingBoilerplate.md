# Упрощение шаблона

Redux частично [вдохновлен Flux](../introduction/PriorArt.md#flux), а самая большая жалоба на Flux — это то, что он заставляет Вас писать много лишнего. В этом рецепте мы рассмотрим, как Redux дает нам выбор, насколько подробно мы хотели бы писать наш код, в зависимости от личного стиля, предпочтений команды, долгосрочном плане ремонтопригодности и так далее.

## Действия

Действия — это простые объекты, описывающее, что происходит в приложении. Они являются единственным способом описать намерение изменить данные. Важно, что **действия, будучи объектами, которые Вы должны отправлять, являются не шаблоном, а одним из [фундаментальных принципов](../introduction/ThreePrinciples.md) Redux**.

Существуют фреймворки, утверждающие, что они подобны Flux, но без концепции объектов действий. С точки зрения предсказуемости, это шаг назад от Flux или Redux. Если в них нет сериализируемых простых объектов действий, невозможно записать и воспроизвести сеансы пользователя, или реализовать [hot reloading with time travel](https://www.youtube.com/watch?v=xsSnOQynTHs). Если Вы предпочитаете изменять данные напрямую, Вам не нужен Redux.

Действия выглядят следующим образом:

```js
{ type: 'ADD_TODO', text: 'Use Redux' }
{ type: 'REMOVE_TODO', id: 42 }
{ type: 'LOAD_ARTICLE', response: { ... } }
```

Общепринято, что действия имеют поле `type`, которое помогает редюсерам (или хранилищам в Flux) опознавать их. Мы [рекомендуем](/docs/faq/Actions.html#почему-тип-действия-должен-быть-строкой-или-по-крайней-мере-сериализуемым-почему-мои-типы-действий-должны-быть-константами) Вам использовать строки (String) вместо [символов (Symbol)](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) для обозначения типа, т.к. строки — сериализуемы, а использование символов может усложнить запись и воспроизведение, когда это потребуется.

Во Flux традиционно считается, что Вы должны определять каждый тип действи строковой константой:

```js
const ADD_TODO = 'ADD_TODO'
const REMOVE_TODO = 'REMOVE_TODO'
const LOAD_ARTICLE = 'LOAD_ARTICLE'
```

Чем это выгодно? **Часто утверждают, что константы не нужны, и для маленьких проектов это может быть правильно.** Для больших проектов, существует несколько преимуществ определение типов действий через константы:

* Это помогает держать наименование последовательным, потому что все типы действий собраны в одном месте.
* Иногда лучше видеть все существующие действия перед началом работы над следующим функционалом. Может такое случиться, что нужное Вам действие уже добавлено кем-то из членов команды, но Вы не знали.
* Список типов действий, которые уже были добавлены, удалены и изменены в Pull Request поможет каждому члену команды отслеживать объем и реализацию нового функционала.
* Если Вы допустили опечатку при импорте константы, Вы получите  `undefined`. Redux немедленно пробросит это, когда будет отправлять такое действие, и Вы найдете ошибку быстрее.

Выбор соглашений для Вашего приложения остается на Ваше усмотрение. Вы можете начать использовать строки, а позже перейти к константам, а, возможно, еще позже — вынести их в отдельный файл. Redux не имеет ограничений по этому поводу, так что это остается на Ваше усмотрение.

## Генераторы действий

Другое общепринятое соглашение — это вместо создания объектов действий в той же части приложения, где эти действия вызываются, создавать функции, генерирующие их.

Например, вместо вызова `dispatch` с агрументом-объектом:

```js
// somewhere in an event handler
dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
```

Вы можете написать генератор действий в отдельном файле и импортировать его в Ваш компонент:

#### `actionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}
```

#### `AddTodo.js`

```js
import { addTodo } from './actionCreators'

// somewhere in an event handler
dispatch(addTodo('Use Redux'))
```

Генераторы действий часто подвергались критике за свою шаблонность. Что ж, Вы можете их не писать! **Вы можете использовать объекты, если чувствуете, что это лучше подходит для Вашего проекта.** Однако, некоторые рекомендации по написанию генераторов действий Вы должны знать.

Допустим, дизайнер приходит к нам после просмотра нашего прототипа и говорит, что надо ограничить число задач тремя максимум. Мы можем реализовать это, переписав наш генератор действий на коллбэк, используя [redux-thunk](https://github.com/gaearon/redux-thunk) миддлвэр и добавив преждевременный выход из функции:

```js
function addTodoWithoutCheck(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function addTodo(text) {
  // This form is allowed by Redux Thunk middleware
  // described below in “Async Action Creators” section.
  return function (dispatch, getState) {
    if (getState().todos.length === 3) {
      // Exit early
      return
    }

    dispatch(addTodoWithoutCheck(text))
  }
}
```

Мы всего лишь изменили поведение генератора действия `addTodo`, совершенно незаметно для кода, использующего этот генератор кода. **Нам не пришлось заботиться о поиске каждого кусочка кода, где происходит добавление, чтобы убедиться, что они работают правильно.** Генераторы действий позволяют Вам отделять дополнительную логику отправки действий от реального выбрасывания этих действий компонентами.

### Создание генераторов действий

Некоторые фреймворки, такие как [Flummox](https://github.com/acdlite/flummox), создают константы типов действий автоматически из описаний функций-генераторов действий. Идея состоит в том, что Вам не надо описывать и `ADD_TODO` константу, и `addTodo()` генератор действий. Под капотом такие решения все еще создают константы типов действий, но они созданы неявно, так что это новый уровень абстракции, что может привести к путанице. Мы рекомендуем Вам создавать константы типов действий самостоятельно.

Написание простого генератора действий может быть утомительно и часто заканчивается созданием избыточно шаблонного кода:

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function editTodo(id, text) {
  return {
    type: 'EDIT_TODO',
    id,
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

Вы также можете написать функцию, которая создает генератор действий:

```js
function makeActionCreator(type, ...argNames) {
  return function(...args) {
    let action = { type }
    argNames.forEach((arg, index) => {
      action[argNames[index]] = args[index]
    })
    return action
  }
}

const ADD_TODO = 'ADD_TODO'
const EDIT_TODO = 'EDIT_TODO'
const REMOVE_TODO = 'REMOVE_TODO'

export const addTodo = makeActionCreator(ADD_TODO, 'todo')
export const editTodo = makeActionCreator(EDIT_TODO, 'id', 'todo')
export const removeTodo = makeActionCreator(REMOVE_TODO, 'id')
```

Также существуют служебные библиотеки для помощи в создании генераторов действий, такие как [redux-act](https://github.com/pauldijou/redux-act) и [redux-actions](https://github.com/acdlite/redux-actions). Они могут помочь уменьшить шаблонный код и обеспечить соблюдение стандартов, таких как [Flux Standard Action (FSA)](https://github.com/acdlite/flux-standard-action).

## Асинхронные генераторы действий

[Миддлвэры (middleware)](../Glossary.md#middleware) позволяют Вам внедрять обычную логику, которая обрабатывает каждое действие перед тем как отправить. Асинхронные действия — наиболее распространенные вариант применения миддлвэров.

Без миддлвэра, [`dispatch`](../api/Store.md#dispatch) принимает только простой объект, поэтому нам приходится выполнять AJAX-запросы внутри компонентов:

#### `actionCreators.js`

```js
export function loadPostsSuccess(userId, response) {
  return {
    type: 'LOAD_POSTS_SUCCESS',
    userId,
    response
  }
}

export function loadPostsFailure(userId, error) {
  return {
    type: 'LOAD_POSTS_FAILURE',
    userId,
    error
  }
}

export function loadPostsRequest(userId) {
  return {
    type: 'LOAD_POSTS_REQUEST',
    userId
  }
}
```

#### `UserInfo.js`

```js
import { Component } from 'react'
import { connect } from 'react-redux'
import { loadPostsRequest, loadPostsSuccess, loadPostsFailure } from './actionCreators'

class Posts extends Component {
  loadData(userId) {
    // Injected into props by React Redux `connect()` call:
    let { dispatch, posts } = this.props

    if (posts[userId]) {
      // There is cached data! Don't do anything.
      return
    }

    // Reducer can react to this action by setting
    // `isFetching` and thus letting us show a spinner.
    dispatch(loadPostsRequest(userId))

    // Reducer can react to these actions by filling the `users`.
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch(loadPostsSuccess(userId, response)),
      error => dispatch(loadPostsFailure(userId, error))
    )
  }

  componentDidMount() {
    this.loadData(this.props.userId)
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.loadData(nextProps.userId)
    }
  }

  render() {
    if (this.props.isFetching) {
      return <p>Loading...</p>
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    )

    return <div>{posts}</div>
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts)
```

Однако, код быстро становится повторяющимся, потому что разные компоненты запрашивать данные из одной и той же точки входа в API. Более того, мы хотим переиспользовать некоторые части этой логики (например, ранний выход, когда доступны кэшированные данные) из многих компонентов.

**Миддлвэр позволяет нам писать более выразительные, потенциально асинхронные генераторые действий.** Это позволяет нам отправлять в качестве действий что-то помимо простых объектов и интерпретировать значения. Например, миддлвэр может “отлавливать” отправленные промисы (Promises) и обращать их в пару из запроса и успешных/провальных действий.

Простейший пример миддлвэра — [redux-thunk](https://github.com/gaearon/redux-thunk). **“Thunk” миддлвэр позволяет нам писать генераторы действий как “thunks” — функции, возвращающие функции.** Это переворачивает управление: Вы будете получать `dispatch` в качестве аргумента, так Вы сможете писать генератор действия, который отправляется несколько раз.

>##### Запомните

>Thunk миддлвэр — всего лишь один пример миддлвэров. Миддлвэр не несет в себе смысл “предоставлять отправку функций”. Он предоставляет Вам возможность отправлять что угодно, что Вы сможете обработать. Thunk миддлвэр добавляет специфическое поведение, когда Вы отправляете функции, но это зависит от используемого миддлвэра.

Рассмотрим приведенный выше код, переписанный с использованием [redux-thunk](https://github.com/gaearon/redux-thunk):

#### `actionCreators.js`

```js
export function loadPosts(userId) {
  // Interpreted by the thunk middleware:
  return function (dispatch, getState) {
    let { posts } = getState()
    if (posts[userId]) {
      // There is cached data! Don't do anything.
      return
    }

    dispatch({
      type: 'LOAD_POSTS_REQUEST',
      userId
    })

    // Dispatch vanilla actions asynchronously
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch({
        type: 'LOAD_POSTS_SUCCESS',
        userId,
        response
      }),
      error => dispatch({
        type: 'LOAD_POSTS_FAILURE',
        userId,
        error
      })
    )
  }
}
```

#### `UserInfo.js`

```js
import { Component } from 'react'
import { connect } from 'react-redux'
import { loadPosts } from './actionCreators'

class Posts extends Component {
  componentDidMount() {
    this.props.dispatch(loadPosts(this.props.userId))
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.props.dispatch(loadPosts(nextProps.userId))
    }
  }

  render() {
    if (this.props.isFetching) {
      return <p>Loading...</p>
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    )

    return <div>{posts}</div>
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts)
```

Так гораздо меньше кода! Если хотите, Вы можете все еще использовать “голые” генераторы действий, такие как `loadPostsSuccess`, которые Вы бы использовали в контейнере `loadPosts` генератора действий.

**В конце концов, Вы можете написать свой собственный миддлвэр.** Допустим, Вы хотите обощить шаблон выше и описывать Ваши асинхронные генераторы действий следующим образом:

```js
export function loadPosts(userId) {
  return {
    // Types of actions to emit before and after
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    // Check the cache (optional):
    shouldCallAPI: (state) => !state.posts[userId],
    // Perform the fetching:
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    // Arguments to inject in begin/end actions
    payload: { userId }
  }
}
```

Миддлвэр, который обрабатывает такие действия, может выглядить так:

```js
function callAPIMiddleware({ dispatch, getState }) {
  return next => action => {
    const {
      types,
      callAPI,
      shouldCallAPI = () => true,
      payload = {}
    } = action

    if (!types) {
      // Normal action: pass it on
      return next(action)
    }

    if (
      !Array.isArray(types) ||
      types.length !== 3 ||
      !types.every(type => typeof type === 'string')
    ) {
      throw new Error('Expected an array of three string types.')
    }

    if (typeof callAPI !== 'function') {
      throw new Error('Expected callAPI to be a function.')
    }

    if (!shouldCallAPI(getState())) {
      return
    }

    const [ requestType, successType, failureType ] = types

    dispatch(Object.assign({}, payload, {
      type: requestType
    }))

    return callAPI().then(
      response => dispatch(Object.assign({}, payload, {
        response,
        type: successType
      })),
      error => dispatch(Object.assign({}, payload, {
        error,
        type: failureType
      }))
    )
  }
}
```

Один раз вызвав [`applyMiddleware(...middlewares)`](../api/applyMiddleware.md), Вы можете писать все Ваши генераторы действий с API вызовами следующим образом:

```js
export function loadPosts(userId) {
  return {
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    shouldCallAPI: (state) => !state.posts[userId],
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    payload: { userId }
  }
}

export function loadComments(postId) {
  return {
    types: ['LOAD_COMMENTS_REQUEST', 'LOAD_COMMENTS_SUCCESS', 'LOAD_COMMENTS_FAILURE'],
    shouldCallAPI: (state) => !state.comments[postId],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`),
    payload: { postId }
  }
}

export function addComment(postId, message) {
  return {
    types: ['ADD_COMMENT_REQUEST', 'ADD_COMMENT_SUCCESS', 'ADD_COMMENT_FAILURE'],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`, {
      method: 'post',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ message })
    }),
    payload: { postId, message }
  }
}
```

## Редюсеры

Redux значительно упрощает шаблон Flux-хранилища описанием логики обновления в виде функции. Функция проще, чем объект, и гораздо проще, чем класс.

Вглянем на это Flux-хранилище:

```js
let _todos = []

const TodoStore = Object.assign({}, EventEmitter.prototype, {
  getAll() {
    return _todos
  }
})

AppDispatcher.register(function (action) {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      let text = action.text.trim()
      _todos.push(text)
      TodoStore.emitChange()
  }
})

export default TodoStore
```

С Redux та же логика обновления может быть описана как упрощенная функция:

```js
export function todos(state = [], action) {
  switch (action.type) {
  case ActionTypes.ADD_TODO:
    let text = action.text.trim()
    return [ ...state, text ]
  default:
    return state
  }
}
```

`Switch` оператор — это *не* шаблон. Реальный шаблон Flux абстрактен: необходимо выдавать обновления данных, необходимо зарегистрировать Хранилище в Диспетчере, необходимо представлять Хранилище в виде объекта (и другие затруднения, которые возникают, когда Вы хотите получить универсальное приложение).

Печально, что многие все еще выбирают Flux-фреймворк на основе того, диктует ли он в документации использовать `switch` оператор. Если Вы не любите `switch`, Вы можете использовать функцию, которую мы показывали прежде.

### Создание редюсеров

Давайте напишем функцию, которая позволит нам выражать редюсеры как объект, сопоставляющий типы действий обработчикам. Например, если мы хотим описывать наши редюсеры для `todos` так:

```js
export const todos = createReducer([], {
  [ActionTypes.ADD_TODO](state, action) {
    let text = action.text.trim()
    return [ ...state, text ]
  }
})
```

То мы можем написать следующий хелпер для достижения этого:

```js
function createReducer(initialState, handlers) {
  return function reducer(state = initialState, action) {
    if (handlers.hasOwnProperty(action.type)) {
      return handlers[action.type](state, action)
    } else {
      return state
    }
  }
}
```

Это было нетрудно, не правда ли? Redux не обеспечивает такие функции-хелперы по умолчанию, потому что существует слишком много способов написать их. Возможно, Вам захочется автоматически конвертировать простые JS-объекты в иммутабельные для «упразднения» состояния сервера. Возможно, Вы захотите объединять возвращаемое состояние с текущим. Может быть много подходов для “отлова всех” обработчиков. Все они зависят от соглашений, которые Вы выберите для Вашей команды и проекта.

API редюсеров Redux заключается в том, что `(state, action) => state`, но то, как Вы это реализуете, решать Вам.
