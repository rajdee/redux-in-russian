# Редьюсеры (Reducers)

**Редьюсеры** определяют, как состояние приложения изменяется в ответ на [экшены](./Actions.md), отправленные в стор. Помните, что экшены только описывают, _что произошло, но не описывают, как изменяется состояние приложения.

## Проектирование структуры состояния (State)

В Redux все состояние приложения хранится в виде единственного объекта. Подумать о его структуре перед написанием кода — довольно неплохая идея. Каково минимальное представление состояния Вашего приложения в виде объекта?

Для нашего todo-приложения, мы хотим хранить две разные сущности:

* Состояние фильтра видимости;
* Актуальный список todo-задач.

Часто вы будете понимать, что вам нужно хранить некоторые данные, а также некоторые состояния пользовательского интерфейса в дереве состояний. Это нормально, только старайтесь такие данные не смешивать с данными, которые описывают состояние UI.

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

>##### Заметка об отношениях

>В более сложном приложении вы, скорее всего, будете иметь разные сущности, которые будут ссылаться друг на друга. Мы советуем поддерживать состояние (state) в настолько упорядоченном виде, насколько это возможно. Старайтесь не допускать никакой вложенности. Держите каждую сущность в объекте, который хранится с ID в качестве ключа. Используйте этот ID в качестве ссылки из других сущностей или списков. Думайте о состоянии приложения (app state), как о базе данных. Этот подход детально описан в [документации к normalizr](https://github.com/gaearon/normalizr). Например, в реальном приложении хранение хеша todo-сущностей `todosById: { id -> todo }` и массива их ID `todos: array<id>` в состоянии (state) было бы лучшей идеей, но мы оставим пример простым.

## Обработка экшенов

Теперь, когда мы определились с тем, как должны выглядеть наши объекты состояния (state objects), мы готовы написать редьюсер для них. Редьюсер (reducer) — это чистая функция, которая принимает предыдущее состояние и экшен (state и action) и возвращает следующее состояние (новую версию предыдущего).

```js
(previousState, action) => newState
```

Функция называется редьюсером (reducer) потому, что ее можно передать в [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). Очень важно, чтобы редьюсеры оставались чистыми функциями. Вот список того, чего **никогда** нельзя делать в редьюсере:

* Непосредственно изменять то, что пришло в аргументах функции;
* Выполнять какие-либо сайд-эффекты: обращаться к API или осуществлять переход по роутам;
* Вызывать не чистые функции, например `Date.now()` или `Math.random()`.

Мы рассмотрим способы выполнения сайд-эффектов в [продвинутом руководстве](../advanced/README.md). На данный момент просто запомните, что редьюсер должен быть чистым. **Получая аргументы одного типа, редьюсер должен вычислять новую версию состояния и возвращать ее. Никаких сюрпризов. Никаких сайд-эффектов. Никаких обращений к стороннему API. Никаких изменений (mutations). Только вычисление новой версии состояния.**

Исходя из вышенаписанных принципов, давайте начнем писать редьюсер, постепенно обучая его понимать [экшены (actions)](Actions.md), которые мы описали чуть раньше.

Мы начнем с определения начального состояния (initial state). В первый раз Redux вызовет редьюсер с неопределенным состоянием(`state === undefined`). Это наш шанс инициализировать начальное состояние приложения:

```js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // Пока не обрабатываем никаких экшенов
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state
}
```

Использование [синтаксиса аргументов по умолчанию из ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Default_parameters) для более компактного написания — просто аккуратный трюк:

```js
function todoApp(state = initialState, action) {
  // Пока не обрабатываем никаких экшенов
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state
}
```

Теперь давайте начнем обрабатывать экшен `SET_VISIBILITY_FILTER`. Все, что нужно сделать — это изменить `visibilityFilter` в состоянии приложения. Это просто:

```js
import {
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'

...

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

Обратите внимание:

1. **Мы не изменяем `state`**. Мы создаем копию с помощью [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).
`Object.assign(state, { visibilityFilter: action.filter })` тоже неверный вариант: в этом случае первый аргумент будет изменен.
Вы **должны** передать первым аргументом пустой объект. Вы также можете подключить [object spread operator proposal](../recipes/UsingObjectSpreadOperator.md), чтобы вместо этого писать `{ ...state, ...newState }` .

2. **Мы возвращаем предыдущую версию состояния (`state`) в `default` ветке.** Очень важно возвращать предыдущую версию состояния (`state`) для любого неизвестного/необрабатываемого экшена (`action`).


>##### Обратите внимание на `Object.assign`

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) это часть ES6, но этот метод не поддерживается старыми браузерами. Вам нужно будет использовать полифилл, [плагин для Babel](https://www.npmjs.com/package/babel-plugin-transform-object-assign), либо хелпер из другой библиотеки, к примеру [`_.assign()` из lodash](https://lodash.com/docs#assign).

>##### Обратите внимание на `switch` и шаблон (boilerplate)

> Конструкция `switch` *не является* реальным требованием. Реальный шаблон Flux определяется концепцией: необходимость инициировать обновление, необходимость зарегистрировать стор (`Store`) в `Dispatcher'е`, необходимость, чтобы `Store` был объектом (возникают осложнения, если вы хотите универсальное приложение (universal app)). Redux решает эти проблемы благодаря использованию чистых редьюсеров вместо генераторов событий (event emitters)

>Если вам не нравится конструкция `switch`, вы можете использовать собственную функцию `createReducer`, которая принимает объект обработчиков, как показано в [“упрощение шаблона (reducing boilerplate)”](../recipes/ReducingBoilerplate.md#reducers).

## Обрабатываем больше экшенов

У нас есть еще два экшена, которые должны быть обработаны! Так же, как мы сделали с `SET_VISIBILITY_FILTER` мы имортируем `ADD_TODO` и `TOGGLE_TODO` экшены и затем допишем наш редьюсер для обработки `ADD_TODO`.

```js
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'

...

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

Как и раньше, мы никогда не изменяем непосредственно `state` или его поля. Вместо этого мы возвращаем новый объект. Новый `todos` равен старому `todos`, в конец которого добавлен новый элемент `todo`. Свежий  tod  был создан с использованием информации, полученной из `action`.

Ну и наконец, имплементация обработчика для экшена `TOGGLE_TODO` не должна стать для Вас большим сюрпризом:

```js
case TOGGLE_TODO:
  return Object.assign({}, state, {
    todos: state.todos.map((todo, index) => {
      if (index === action.index) {
        return Object.assign({}, todo, {
          completed: !todo.completed
        })
      }
      return todo
    })
  })
```

Поскольку мы хотим обновить конкретный элемент в массиве, не прибегая к мутациям, мы должны создать новый массив с теми же элементами, за исключением элемента по индексу. Если вы часто пишете такие операции, рекомендуется использовать хэлперы, такие как [immutability-helper] (https://github.com/kolodny/immutability-helper), [updeep] (https://github.com/essential/updeep) или даже такую библиотеку, как [Immutable] (http://facebook.github.io/immutable-js/), которая имеет встроенную поддержку для глубоких обновлений. Просто запомните, что нельзя присваивать ничего внутри `state`, пока вы его не склонировали.

## Разделение редьюсеров

Вот так выглядит наш код на данный момент. Выглядит излишне многословным:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if (index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

Есть ли способ облегчить понимание? Кажется, что `todos` и `visibilityFilter` обновляются совершенно независимо. Иногда поля состояния (state fields) зависят от других полей и требуется б*о*льшая связанность, но в нашем случаем мы безболезненно можем вынести обновление `todos` в отдельную функцию:

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

Обратите внимание, что функция `todos` также принимает `state`, но `state` — это массив! Теперь `todoApp` просто передает срез состояния в функцию `todos`, которая, свою очередь, точно знает, как обновить именно этот кусок состояния. **Это называется  *композицией редьюсеров* и является фундаментальным шаблоном построения Redux-приложений.** 

Давайте рассмотрим композицию редьюсеров подробнее. Можем ли мы извлечь редьюсер, который будет управлять только `visibilityFilter`? Конечно можем:

Ниже нашего импорта давайте использовать [ES6 Object Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), чтобы объявить `SHOW_ALL`:

```
const { SHOW_ALL } = VisibilityFilters
```

Затем:

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}
```

Теперь мы можем переписать наш главный редьюсер в виде функции, которая вызывает другие редьюсеры, обрабатывающие части состояния и собирает отдельно обработанные части состояния в один цельный объект.
Также главному редьюсеру больше нет необходимости знать полное начальное состояние. Достаточно того, что каждый дочерний редьюсер возвращает свое начальное состояние, если при первом вызове получает `undefined` вместо `state`.

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

**Обратите внимание на то, что каждый из этих дочерних редьюсеров управляет только какой-то одной частью глобального состояния. Параметр `state` разный для каждого отдельного дочернего редьюсера и соответствует той части глобального состояния, которой управляет этот дочерний редьюсер.**

Уже выглядит лучше! Когда приложение разрастается, мы можем выносить редьюсеры в отдельные файлы и поддерживать их совершенно независимыми, что дает нам возможность управлять различными разделами наших данных.

Наконец, Redux предоставляет утилиту, называемую [`combineReducers()`](../api/combineReducers.md), которая реализует точно такой же логический шаблон, который мы только что реализовали в `todoApp`. С ее помощью мы можем переписать `todoApp` следующим образом:

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

Обратите внимание, что это полностью эквивалентно такому коду:

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

Вы также можете назначать им разные ключи или вызывать функции по-разному. Есть два совершенно равноценных способа писать комбинированные редьюсеры:

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

Все, что делает [`combineReducers()`](../api/combineReducers.md) — это генерирует функцию, которая вызывает ваши редьюсеры c **частью глобального состояния, которая выбирается в соответствии с их ключами**, и затем снова собирает результаты всех вызовов в один объект. [Тут нет никакой магии.](https://github.com/reactjs/redux/issues/428#issuecomment-129223274) И, как и другие редьюсеры,  `combineReducers()` не создает новый объект, если все предоставленные ему редьюсеры не изменяют состояние.

>##### Заметка для сообразительных пользователей синтаксиса ES6

>Т.к. `combineReducers` ожидает на входе объект, мы можем поместить все редьюсеры верхнего уровня в разные файлы, экспортировать каждую функцию-редьюсер и использовать `import * as reducers` для получения их в формате объекта, ключами которого будут имена экспортируемых функций.

>```js
>import { combineReducers } from 'redux'
>import * as reducers from './reducers'
>
>const todoApp = combineReducers(reducers)
>```
>
>Поскольку `import *` — это все еще новый синтаксис, мы не используем его нигде в документации во избежание [путаницы](https://github.com/reactjs/redux/issues/428#issuecomment-129223274), но вы можете случайно наткнуться на него в каких-нибудь примерах кода из сообщества.

## Исходный код

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## Следующие шаги

Далее мы изучим как [создать Redux-стор](Store.md), которое содержит состояние и заботится о вызове редьюсеров, когда вы отправляете экшен.

