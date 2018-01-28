# Вычисление производных данных

[Reselect](https://github.com/reactjs/reselect) это простая библиотека для создания мемоизированных, пригодных для компоновки **селекторных** функций. Селекторы Reselect могут использоваться для эффективного вычисления производных данных из Redux store.

### Причины использовать Мемоизированные Селекторы

Давайте вспомним наш Список Задач [пример Todos List](../basics/UsageWithReact.md):

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

В приведённом выше примере, `mapStateToProps` вызывает `getVisibleTodos` чтобы посчитать `todos`. Это отлично работает, но есть недостаток: `todos` расчитывается каждый раз, когда компонент обновляется. Если дерево состояний велико, или вычисление требует больших затрат, повторение вычисления при каждом обновлениии может привести к проблемам с производительностью. Reselect может помочь избежать этих излишних пересчётов.

### Создание Мемоизированного Селектора

Мы хотели бы заменить `getVisibleTodos` на мемоизированный селектор, который пересчитывает `todos` когда значение `state.todos` или `state.visibilityFilter` изменяется, но не тогда когда изменения происходят в других (независимых) частях дерева состояний.

Reselect предоставляет функцию `createSelector` для создания мемоизированных селекторов. В качестве аргументов `createSelector`принимает массив входных селекторов и функцию преобразования. Если дерево состояний Redux мутируется таким образом, что послужит причиной изменения значения входного селектора, селектор вызовет свою функцию преобразования со значениями входных селекторов в качестве аргументов и вернёт результат. Если значения входных селекторов такие же как и в предыдущем вызове селектора, он вернёт ранее вычисленное значение, вместо того чтобы вызывать функцию преобразования.

Давайте определим мемоизированный селектор с именем `getVisibleTodos` на замену не мемоизированной версии выше:

#### `selectors/index.js`

```js
import { createSelector } from 'reselect'

const getVisibilityFilter = (state) => state.visibilityFilter
const getTodos = (state) => state.todos

export const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)
```

В примере выше, `getVisibilityFilter` и `getTodos` входные селекторы. Они создаются как обычные не мемоизированные селекторные функции, потому что они не преобразуют данные, которые они выбирают. Что же касается `getVisibleTodos`- это мемоизированный селектор. Он принимает `getVisibilityFilter` и `getTodos` в качестве входных селекторов, и функцию преобразования, которая вычисляет отфильтрованный список задач (todos list).

### Композиция Селекторов

Мемоизированный селектор сам по себе может быть входным селектором для другого мемоизированного селектора. Здесь `getVisibleTodos` используется в качестве входного селектора для селектора, который затем фильтрует todos по ключевому слову:

```js
const getKeyword = (state) => state.keyword

const getVisibleTodosFilteredByKeyword = createSelector(
  [ getVisibleTodos, getKeyword ],
  (visibleTodos, keyword) => visibleTodos.filter(
    todo => todo.text.indexOf(keyword) > -1
  )
)
```

### Подключение Селектора к Redux Store

Если Вы используете [React Redux](https://github.com/reactjs/react-redux), Вы можете вызывать селекторы в качестве регулярных функций внутри `mapStateToProps()`:

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'
import { getVisibleTodos } from '../selectors'

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

### Доступ к React Props в Селекторах

> В этом разделе предоставлено гипотетическое расширение нашего приложения, которое позволяет ему поддерживать любое количество списков задач (Todo Lists). Пожалуйста, обратите внимание, полная реализация этого расширения требует изменений в редюсерах (reducers), компонентах (components), действиях (actions) и т.д., которые не имеют прямого отношения к обсуждаемым темам и для краткости были опущены.

До сих пор мы видели что селекторы получают состояние хранилища (store state) Redux в качестве аргумента, но селектор также может получать props.

Вот компонент `App` который отображает три `VisibleTodoList` компонента, каждый из которых имеет `listId` prop:

#### `components/App.js`

```js
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <VisibleTodoList listId="1" />
    <VisibleTodoList listId="2" />
    <VisibleTodoList listId="3" />
  </div>
)
```

Каждый `VisibleTodoList` контейнер должен выбирать различный срез состояния (state) в зависимости от значения `listId` prop, поэтому давайте модифицируем `getVisibilityFilter` и `getTodos` для приёма аргумента props:

#### `selectors/todoSelectors.js`

```js
import { createSelector } from 'reselect'

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter

const getTodos = (state, props) =>
  state.todoLists[props.listId].todos

const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_COMPLETED':
        return todos.filter(todo => todo.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(todo => !todo.completed)
      default:
        return todos
    }
  }
)

export default getVisibleTodos
```

`props` может быть передан `getVisibleTodos` из `mapStateToProps`:

```js
const mapStateToProps = (state, props) => {
  return {
    todos: getVisibleTodos(state, props)
  }
}
```

Итак, теперь `getVisibleTodos` имеет доступ к `props`, и всё кажется работает нормально.

**Но есть проблема!**

Использование селектора `getVisibleTodos` с множественными вхождениями контейнера `visibleTodoList` не будет правильно мемоизировано:

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'
import { getVisibleTodos } from '../selectors'

const mapStateToProps = (state, props) => {
  return {
    // WARNING: THE FOLLOWING SELECTOR DOES NOT CORRECTLY MEMOIZE
    todos: getVisibleTodos(state, props)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

Селектор созданный с помощью `createSelector` возвращает только кэшированное значение, когда его набор аргументов совпадает с его предыдущим набором аргументов. Если мы рендерим поочерёдно `<VisibleTodoList listId="1" />` и `<VisibleTodoList listId="2" />`, общий селектор будет поочерёдно принимать `{listId: 1}` и `{listId: 2}` как аргумент `props`. Это приведёт к тому что аргументы будут разными для каждого вызова, поэтому селектор всегда будет пересчитывать, вместо того чтобы возвращать кэшированное значение. Мы увидим как преодолеть это ограничение в следующем разделе.

### Совместное использование селекторов с несколькими компонентами

> Примеры в этом разделе требуют React Redux v4.3.0 или выше

Чтобы совместно использовать селектор для нескольких компонентов `VisibleTodoList` **и** сохранять мемоизацию, каждому экземпляру компонента нужна собственная личная копия селектора.

Давайте создадим функцию `makeGetVisibleTodos`, которая возвращает новую копию селектора `getVisibleTodos` при каждом вызове:

#### `selectors/todoSelectors.js`

```js
import { createSelector } from 'reselect'

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter

const getTodos = (state, props) =>
  state.todoLists[props.listId].todos

const makeGetVisibleTodos = () => {
  return createSelector(
    [ getVisibilityFilter, getTodos ],
    (visibilityFilter, todos) => {
      switch (visibilityFilter) {
        case 'SHOW_COMPLETED':
          return todos.filter(todo => todo.completed)
        case 'SHOW_ACTIVE':
          return todos.filter(todo => !todo.completed)
        default:
          return todos
      }
    }
  )
}

export default makeGetVisibleTodos
```

Нам также нужен способ предоставить каждому экземпляру контейнера доступ к его собственному селектору. Аргумент `mapStateToProps` от `connect` может помочь в этом.

**Если аргумент `mapStateToProps` предоставленный `connect` возвращает функцию вместо объекта, он будет использоваться для создания отдельной функции `mapStateToProps` для каждого экземпляра контейнера.**

В приведённом ниже примере `makeMapStateToProps` создаёт новый `getVisibleTodos` селектор, и возвращает функцию `mapStateToProps`, которая имеет эксклюзивный доступ к новому селектору:

```js
const makeMapStateToProps = () => {
  const getVisibleTodos = makeGetVisibleTodos()
  const mapStateToProps = (state, props) => {
    return {
      todos: getVisibleTodos(state, props)
    }
  }
  return mapStateToProps
}
```

Если мы передадим `makeMapStateToProps`  `connect`, каждый экземпляр контейнера `VisibleTodosList` получит свою собственную функцию `mapStateToProps` с собственным селектором `getVisibleTodos`. Мемоизация теперь будет работать правильно, независимо от порядка отрисовки (рендера) контейнеров `VisibleTodoList`.

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'
import { makeGetVisibleTodos } from '../selectors'

const makeMapStateToProps = () => {
  const getVisibleTodos = makeGetVisibleTodos()
  const mapStateToProps = (state, props) => {
    return {
      todos: getVisibleTodos(state, props)
    }
  }
  return mapStateToProps
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  makeMapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

## Следующие шаги

Ознакомьтесь с [официальной документацией](https://github.com/reactjs/reselect) Reselect а также [FAQ](https://github.com/reactjs/reselect#faq). Большинство проектов Redux начинают использовать Reselect когда у них возникают проблемы с производительностью из-за слишком большого количества вторичных вычислений и потерь в ре-рендеринге, поэтому убедитесь, что вы знакомы с ним, прежде чем создавать что-то большое. Также может быть полезно изучить [его исходный код](https://github.com/reactjs/reselect/blob/master/src/index.js) чтобы вы не думали что это волшебство.
