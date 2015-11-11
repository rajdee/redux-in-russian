# Редьюсеры (Reducers)

[Действия (Actions)](./Actions.md) описывают тот факт, что *что-то совершилось*, но не определяют, как в ответ изменяется состояние (state) приложения. Это работа для редьюсеров (reducers).

## Проектирование структуры состояния (State)

В Redux, все состояние приложения хранится в виде единственного объекта. Подумать о его структуре перед написанием кода - довольно неплохая идея. Каково минимальное представление состояния Вашего приложения в виде объекта?

Для нашего ToDo приложения мы хотим хранить две разные сущности:

* Состояние фильтра видимости;
* Актуальный список todo-задач.

Часто Вы будете понимать, что в дереве состояний (state tree) нужно хранить какие-то данные, которые не совсем относятся к состоянию UI. Это нормально, только старайтесь такие данные не смешивать с данными, которые описывают состояние UI.

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

>В более сложном приложении Вы, скорее всего, будете иметь разные сущности, которые будут ссылаться друг на друга. Мы советуем поддерживать состояние (state) в настолько упорядоченном виде, насколько это возможно. Старайтесь не допускать никакой вложенности. Держите каждую сущность в объекте, который хранится с ID в качестве ключа. Используйте этот ID в качестве ссылки из других сущностей или списков. Думайте о состоянии приложения (apps state), как о базе данных. Этот подход детально описан в [документации к normalizr](https://github.com/gaearon/normalizr). Например, в реальном приложении хранение хеша ToDo сущностей `todosById: { id -> todo }` и массива их ID `todos: array<id>` в состоянии (state) было бы лучшей идеей, но мы оставим пример простым.

## Обработка действий (Handling Actions)

Теперь, когда мы определились с тем, как должны выглядеть наши объекты состояния (state objects), мы готовы написать редьюсер для них. Редьюсер (reducer) - это чистая функция, которая принимает предыдущее состояние и действие (state и action) и возвращает следующее состояние (новую версию предыдущего).

```js
(previousState, action) => newState
```

Функция называется редьюсером (reducer) потому, что ее можно передать в [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). Очень важно, чтобы редьюсеры оставались чистыми функциями. Вот список того, чего **никогда** нельзя делать в редьюсере:

* Непосредственно изменять то, что пришло в аргументах функции;
* Выполнять какие-либо сайд-эффекты: обращаться к API или осуществлять переход по роутам;
* Вызывать не чистые функции, например `Date.now()` или `Math.random()`.

Мы рассмотрим способы выполнения сайд-эффектов в [продвинутом руководстве](../advanced/README.md). На данный момент просто запомните, что редьюсер должен быть чистым. **Получая аргументы одного типа, редьюсер должен вычислять новую версию состояния и возвращать ее. Никаких сюрпризов. Никаких сайд-эффектов. Никаких обращений к стороннему API. Никаких изменений (mutations). Только вычисление новой версии состояния.**

Исходя из вышенаписанных принципов, давайте начнем писать редьюсер, постепенно обучая его понимать [действия (actions)](Actions.md), которые мы описали чуть раньше.

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

  // Пока не обрабатываем никаких действий
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state;
}
```

Использование [синтаксиса аргументов по умолчанию из ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) для более компактного написания - просто аккуратный трюк:

```js
function todoApp(state = initialState, action) {
  // Пока не обрабатываем никаких действий
  // и просто возвращаем состояние, которое приняли в качестве параметра
  return state;
}
```

Теперь давайте начнем обрабатывать действие `SET_VISIBILITY_FILTER`. Все, что нужно сделать - это изменить `visibilityFilter` в состоянии приложения. Это просто:

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

1. **Мы не изменяем `state`** Мы, с помощью [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign), создаем обновленную копию `state`. 
`Object.assign(state, { visibilityFilter: action.filter })` тоже не верный вариант. В этом случае первый аргумент будет изменен.
Вы **должны** передать первым аргументом пустой объект. Вы также можете попробовать использовать экспериментальный [object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread), предложенный в ES7. В этом случае можно просто написать `{ ...state, ...newState }`

2. Если в операторе `switch` мы попадаем в `default` ветку, то **возвращаем предыдущую версию состояния** - состояние без изменений. Очень важно возвращать предыдущую версию состояния (`state`) для любого неизвестного/необрабатываемого действия (`action`).


>##### Обратите внимание на `Object.assign`

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) это часть ES6, но этот метод пока не реализован в большинстве браузеров. Вам нужно будет использовать либо полифилл [Babel плагин](https://www.npmjs.com/package/babel-plugin-object-assign), либо хелпер из другой библиотеки, к примеру [`_.assign()` из lodash](https://lodash.com/docs#assign).

>##### Обратите внимание на `switch` и Шаблон (Boilerplate ориг)

> Конструкция `switch` *не является* реальным шаблоном. Реальный шаблон Flux является абстракцией: необходимость инициировать обновление, необходимость зарегистрировать хранилище (`Store`) в `Dispatcher'е`, необходимость, чтобы `Store` был объектом (возникают осложнения, если Вы хотите универсальное приложение (universal app)). Redux решает эти проблемы благодаря использованию чистых редьюсеров вместо генераторов событий (event emitters)

>Если Вам не нравится конструкция `switch`, Вы можете использовать собственную функцию `createReducer`, которая принимает объект обработчиков, как показано в [“сокращении шаблонности (reducing boilerplate)”](../recipes/ReducingBoilerplate.md#reducers).

## Обрабатываем больше действий

У нас есть еще два действия, которые должны быть обработаны! Давайте расширим наш редьюсер так, чтобы он мог обрабатывать действие `ADD_TODO`.

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

Как и раньше, мы никогда не изменяем непосредственно `state` или его поля. Вместо этого мы возвращаем новый объект. В случае добавления новый `todos`, объект - это старый `todos` объект, в данном случае массив, в конец которого добавлен новый элемент `todo`. Свежий `todos` объект был создан с использованием информации, полученной из `action`.

Ну и наконец, имплементация обработчика для действия `COMPLETE_TODO` не должна стать для Вас болшим сюрпризом:

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

В силу того, что мы хотим обновить конкретный `todo` элемент в массиве не прибегая к изменению исходного массива, мы должны использовать функцию slice для массива. Мы как бы создадим новый массив из кусочков старого - `[элементы до интересующего нас элемента, обновленный интересующий нас элемент, элементы после интересующего нас элемента]`. Если Вы поймаете себя на частом написании такого рода решений, то, возможно, Вам стоит обратить внимание на такие утилиты как [React.addons.update](https://facebook.github.io/react/docs/update.html), [updeep](https://github.com/substantial/updeep) или даже такие библиотеки, как [Immutable](http://facebook.github.io/immutable-js/), которые имеют нативную поддержку глубоких обновлений (deep updates). Просто запомните, что нельзя присваивать ничего внутри `state` пока вы его не склонировали.

## Разделение редьюсеров (Splitting Reducers)

Вот так выглядит наш код на данный момент. Выглядит излишне многословным:

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

Есть ли какой-либо способ сделать его более простым для чтения и понимания? Кажется, что `todos` и `visibilityFilter` обновляются совершенно независимо. Иногда поля состояния (state fields) зависят от других полей и требуется б*о*льшая связанность, но в нашем случаем мы безболезненно можем вынести обновление `todos` в отдельную функцию:

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

Обратите внимание, что функция `todos` также принимает `state`, но `state` - это массив! Теперь `todoApp` просто передает срез/слой состояния в функцию `todos`, которая, свою очередь, точно знает, как обновить именно этот/такой кусок состояния. **Это называется *reducer composition* и является фундаментальным шаблоном построения Redux приложений** Мы разбиваем главный редьюсер на дочерние "подредьюсеры", которые выполняют каждый свою работу - обрабатывают какой-то один кусочек данных (срез состояния). А главный редьюсер только лишь решает какому дочернему редьюсеру и какой срез состояния отдать.

Давайте рассмотрим reducer composition подробнее. Можем ли мы извлечь редьюсер, который будет управлять только `visibilityFilter`? Конечно можем:

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

Теперь мы можем переписать наш главный редьюсер, как функцию, которая:
* вызывает другие редьюсеры, обрабатывающие части состояния;
* собирает отдельно обработанные части состояния в один цельный объект.
Также главному редьюсеру больше нет необходимости знать полное начальное состояние. Достаточно того, что каждый дочерний редьюсер возвращает свое начальное состояние, если при первом вызове получает `undefined` вместо `state`.

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

** Обратите внимание на то, что каждый из этих дочерних редьюсеров управляет только какой-то одной/своей частью глобального состояния. Параметр `state` разный для каждого отдельного дочернего редьюсера и соответствует той части глобального состояния, которой управляет этот дочерний редьюсер.**

Уже выглядит лучше! Когда приложение разрастается мы можем выносить редьюсеры в отдельные файлы и поддерживать их совершенно независимыми, что дает нам возможность управлять различными разделами/срезами наших данных.

Наконец, Redux предоставляет утилиту называемую [`combineReducers()`](../api/combineReducers.md), которая реализует точно такой же логический шаблон, который мы только что реализовали в `todoApp`. С ее помощью мы можем переписать `todoApp` следующим образом:

```js
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```

Обратите внимание, что это полностью эквивалентно такому коду:

```js
export default function todoApp(state, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

Есть два совершенно равноценных способа писать комбинированные редьюсеры:

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

Все, что делает [`combineReducers()`](../api/combineReducers.md) - это генерирует функцию, которая вызывает ваши редьюсеры, передавая им в качестве одного из аргументов **срез глобального состояния, который выбирается в соответствии с именем его ключа в глобальном состоянии**, и затем снова собирает результаты всех вызовов в один объект. [Тут нет никакой магии.](https://github.com/rackt/redux/issues/428#issuecomment-129223274)

>##### Заметка для сообразительных пользователей синтаксиса ES6

>Т.к. `combineReducers` ожидает на входе объект, мы можем поместить все редьюсеры верхнего уровня в разные файлы, экспортировать каждую функцию-редьюсер и использовать `import * as reducers` для получения их в формате объекта, ключами которого будут имена экспортируемых функций.

>```js
>import { combineReducers } from 'redux';
>import * as reducers from './reducers';
>
>const todoApp = combineReducers(reducers);
>```
>
>Поскольку `import *` это все еще новый синтаксис, мы не используем его нигде в документации во избежание [путаницы](https://github.com/rackt/redux/issues/428#issuecomment-129223274), но Вы можете случайно наткнуться на него в каих-нибудь примерах кода из сообщества.

## Исходный код (Source Code)

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

## Следующие шаги

Далее мы изучим как [создать Redux хранилище (create a Redux store)](Store.md), которое содержит состояние (state) и заботится о вызове редьюсеров когда Вы генерируете и транслируете действие (dispatch an action).
