# Редьюсеры (Reducers)

[Действия (Actions)](./Actions.md) описывают тот факт, что *что-то совершилось*, но не определяют как в ответ изменяется состояние (state) приложения. Это работа редьюсеров (reducers).

## Проектирование формы состояния (State)

В Redux, все состояния приложения хранится в виде единственного объекта. Подумать о его форме перед написанием кода - довольно не плохая идея. Каково минимальное представление состояния Вашего приложения в виде объекта?

Для нашего ToDo приложения мы хотим хранить две разные вещи:

* Состояние фильтра видимости;
* Актиульный список todo-задач.

Часто Вы будете понимать, что в дереве состояний (state tree) нужно хранить какие то данные, которые не совсем относятся к состоянию UI. Это нормально, только старайтесь держать не смешивать с данными, которые описывают состояние UI.

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [{
    text: 'Consider using Redux',
    completed: true,
  }, {
    text: 'Keep all state in a single tree',
    completed: false
  }]
}
```

>##### Заметка об отношениях

>В более сложном приложении Вы, скорее всего, будете иметь разные сущности, которые будут ссылаться друг на друга. Мы советуем поддерживать состояние (state) в настолько нормализованном виде, насколько это возможно. Старайтесь не допускать никакой вложенности. Держите каждую сущность в объекте, который хранится с ID в качестве ключа. Используйте этот ID в качестве ссылки из других сущностей или списков. Думайте о состоянии приложения (apps state), как о базе данных. Этот подход детально описан в [документации к normalizr](https://github.com/gaearon/normalizr). Например, в реальном приложении хранение хеша ToDo сущностей `todosById: { id -> todo }` и массива их ID `todos: array<id>` в состоянии (state) было бы лучшей идеей, но мы оставим пример простым.

## Обработка действий (Handling Actions)

Теперь, когда мы определились с тем, как должны выглядеть наши объекты состояния (state objects), мы готовы написать редьюсер для них. Редьюсер (reducer) - это чистая функция, котора принимает предыдущее состояние и действие (state и action) и возвращает следующее состояние (новую версию предыдущего).

```js
(previousState, action) => newState
```

Функция называется редьюсером (reducer) потому, что ее можно передать в [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). Очент важно чтобы редьюсеры (reducer) оставались чистыми функциями. Вот список того, чего **никогда** нельзя делать в редьюсере:

* Непосредственно изменять то, что пришло в аргументах функции;
* Выполнять какие-либо сайд-эффекты: обращаться к API или осуществлять переход по роутам;
* Вызывать не чистые функции, например `Date.now()` or `Math.random()`.

Мы рассмотрим способы выполнения сайд-эффектов в [продвинутом руководстве](../advanced/README.md). На данный момент просто запомните, что редьюсер должен быть чистым. **Получая аргументы, которые имеют неизменный тип, редьюсер должен вычислять новую версию состояния и возвращать ее. Никаких сюрпризов. Никаких сайд-эффектов. Никаких обращений к стороннему API. Никаких изменений (mutations). Только вычисление новой версии состояния.**

С осознанием вышенаписанного, давайте начнем писать редьюсер, постепенно обучая его понимать [действия (actions)](Actions.md), которые мы описали чуть раньше.

Мы начнем с определения начального состояния (initial state). В первый раз Redux вызовет редьюсер с неопределенным состоянием(`state === undefined`). Это наш шанс инициализировать начальное состояние приложения:

```js
import { VisibilityFilters } from './actions';

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
};

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState;
  }

  // Пока не обрабатываем никаких жействий
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state;
}
```

Использование [синтаксиса аргументов по умолчанию из ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) для более компактного написания - просто аккуратный трюк:

```js
function todoApp(state = initialState, action) {
  // Пока не обрабатываем никаких жействий
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state;
}
```

Теперь давайте начнем обрабатывать действие `SET_VISIBILITY_FILTER`. Все, что нужно сделать это изменить `visibilityFilter` в состоянии приложения. Это просто:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  default:
    return state;
  }
}
```

Обратите внимание:

1. **Мы не изменяем `state`** Мы, с помощью [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) создаем копию `state`. 
`Object.assign(state, { visibilityFilter: action.filter })` тоже не верный вариант. В этом случае первый аргумент будет изменен.
Вы **должны** передать первым аргументом пустой объект. Вы также можете попробовать использовать экспериментальный [object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread), предложенный в ES7. В этом случае можно просто написать `{ ...state, ...newState }`

2. Если в операторе `switch` мы попадаем в `default` ветку, то **возвращаем предыдущую версию состояния** - состояние без изменений. Очень важно возвращать предыдущую версию состояния (`state`) для любого неизвестного/необрабатываемого действия (`action`).


>##### Обратите внимание на `Object.assign`

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) это часть ES6, но этот метод пока не реализован в большинстве браузеров. Вам нужно будет использовать либо полифилл [Babel плагин](https://www.npmjs.com/package/babel-plugin-object-assign), либо хелпериз другой библиотеки, к примеру [`_.assign()` из lodash](https://lodash.com/docs#assign).

>##### Обратите внимание на `switch` and Шаблонность

> Конструкция `switch` *не является* реальным шаблоном (boilerplate ориг). Реальный шаблон Flux является абстракцией: необходимость инициировать обновление, необходимость зарегистрировать хранилище (`Store`) в `Dispatcher'е`, необходимость, чтобы `Store` был объектом (возникают осложнения, если Вы хотите универсальное приложение (universal app)). Redux решает эти проблемы благодаря использованию читсых редьюсеров вместо генераторов событий (event emitters)

>The `switch` statement is *not* the real boilerplate. The real boilerplate of Flux is conceptual: the need to emit an update, the need to register the Store with a Dispatcher, the need for the Store to be an object (and the complications that arise when you want a universal app). Redux solves these problems by using pure reducers instead of event emitters.

>Если Вам не нравится конструкция `switch`, Вы можете использовать собственную функцию `createReducer`, которая принимает объект обработчиков, как показано в [“сокращении шаблонности (reducing boilerplate)”](../recipes/ReducingBoilerplate.md#reducers).

## Handling More Actions

We have two more actions to handle! Let’s extend our reducer to handle `ADD_TODO`.

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
    return Object.assign({}, state, {
      todos: [...state.todos, {
        text: action.text,
        completed: false
      }]
    });    
  default:
    return state;
  }
}
```

Just like before, we never write directly to `state` or its fields, and instead we return new objects. The new `todos` is equal to the old `todos` concatenated with a single new item at the end. The fresh todo was constructed using the data from the action.

Finally, the implementation of the `COMPLETE_TODO` handler shouldn’t come as a complete surprise:

```js
case COMPLETE_TODO:
  return Object.assign({}, state, {
    todos: [
      ...state.todos.slice(0, action.index),
      Object.assign({}, state.todos[action.index], {
        completed: true
      }),
      ...state.todos.slice(action.index + 1)
    ]
  });
```

Because we want to update a specific item in the array without resorting to mutations, we have to slice it before and after the item. If you find yourself often writing such operations, it’s a good idea to use a helper like [React.addons.update](https://facebook.github.io/react/docs/update.html), [updeep](https://github.com/substantial/updeep), or even a library like [Immutable](http://facebook.github.io/immutable-js/) that has native support for deep updates. Just remember to never assign to anything inside the `state` unless you clone it first.

## Splitting Reducers

Here is our code so far. It is rather verbose:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
    return Object.assign({}, state, {
      todos: [...state.todos, {
        text: action.text,
        completed: false
      }]
    });
  case COMPLETE_TODO:
    return Object.assign({}, state, {
      todos: [
        ...state.todos.slice(0, action.index),
        Object.assign({}, state.todos[action.index], {
          completed: true
        }),
        ...state.todos.slice(action.index + 1)
      ]
    });
  default:
    return state;
  }
}
```

Is there a way to make it easier to comprehend? It seems like `todos` and `visibilityFilter` are updated completely independently. Sometimes state fields depend on one another and more consideration is required, but in our case we can easily split updating `todos` into a separate function:

```js
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
  case COMPLETE_TODO:
    return Object.assign({}, state, {
      todos: todos(state.todos, action)
    });
  default:
    return state;
  }
}
```

Note that `todos` also accepts `state`—but it’s an array! Now `todoApp` just gives it the slice of the state to manage, and `todos` knows how to update just that slice. **This is called *reducer composition*, and it’s the fundamental pattern of building Redux apps.**

Let’s explore reducer composition more. Can we also extract a reducer managing just `visibilityFilter`? We can:

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}
```

Now we can rewrite the main reducer as a function that calls the reducers managing parts of the state, and combines them into a single object. It also doesn’t need to know the complete initial state anymore. It’s enough that the child reducers return their initial state when given `undefined` at first.

```js
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

**Note that each of these reducers is managing its own part of the global state. The `state` parameter is different for every reducer, and corresponds to the part of the state it manages.**

This is already looking good! When the app is larger, we can split the reducers into separate files and keep them completely independent and managing different data domains.

Finally, Redux provides a utility called [`combineReducers()`](../api/combineReducers.md) that does the same boilerplate logic that the `todoApp` above currently does. With its help, we can rewrite `todoApp` like this:

```js
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```

Note that this is completely equivalent to:

```js
export default function todoApp(state, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

You could also give them different keys, or call functions differently. These two ways to write a combined reducer are completely equivalent:

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
});
```

```js
function reducer(state, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  };
}
```

All [`combineReducers()`](../api/combineReducers.md) does is generate a function that calls your reducers **with the slices of state selected according to their keys**, and combining their results into a single object again. [It’s not magic.](https://github.com/rackt/redux/issues/428#issuecomment-129223274)

>##### Note for ES6 Savvy Users

>Because `combineReducers` expects an object, we can put all top-level reducers into a separate file, `export` each reducer function, and use `import * as reducers` to get them as an object with their names as the keys:

>```js
>import { combineReducers } from 'redux';
>import * as reducers from './reducers';
>
>const todoApp = combineReducers(reducers);
>```
>
>Because `import *` is still new syntax, we don’t use it anymore in the documentation to avoid [confusion](https://github.com/rackt/redux/issues/428#issuecomment-129223274), but you may encounter it in some community examples.

## Source Code

#### `reducers.js`

```js
import { combineReducers } from 'redux';
import { ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions';
const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```

## Next Steps

Next, we’ll explore how to [create a Redux store](Store.md) that holds the state and takes care of calling your reducer when you dispatch an action.
