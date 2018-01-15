# Вычисление производных данных

[Reselect](https://github.com/reactjs/reselect) это простая библиотека для создания мемоизированных, пригодных для компоновки **селекторных** функций. Селекторы Reselect могут использоваться для эффективного вычисления полученных данных из Redux store.

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

Reselect представляет функцию `createSelector` для создания мемоизированных селекторов. В качестве аргументов `createSelector`принимает массив входных селекторов и функцию преобразования. Если дерево состояний Redux мутируется таким образом, что послужит причиной изменения значения входного селектора, селектор вызовет свою функцию преобразования со значениями входных селекторов в качестве аргументов и вернёт результат. Если значения входных селекторов такие же как и в предыдущем вызове селектора, он вернёт ранее вычисленное значение вместо вызова функции преобразования.

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

В примере выше, `getVisibilityFilter` и `getTodos` входные селекторы. Они создаются как обычные не мемоизированные селекторные функции потому что они не преобразуют данные, которые они выбирают. Что же касается `getVisibleTodos`- это мемоизированный селектор. Он принимает `getVisibilityFilter` и `getTodos` в качестве входных селекторов, и функцию преобразования, которая вычисляет отфильтрованный список задач (todos list).

### Composing Selectors

A memoized selector can itself be an input-selector to another memoized selector. Here is `getVisibleTodos` being used as an input-selector to a selector that further filters the todos by keyword:

```js
const getKeyword = (state) => state.keyword

const getVisibleTodosFilteredByKeyword = createSelector(
  [ getVisibleTodos, getKeyword ],
  (visibleTodos, keyword) => visibleTodos.filter(
    todo => todo.text.indexOf(keyword) > -1
  )
)
```

### Connecting a Selector to the Redux Store

If you are using [React Redux](https://github.com/reactjs/react-redux), you can call selectors as regular functions inside `mapStateToProps()`:

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

### Accessing React Props in Selectors

> This section introduces a hypothetical extension to our app that allows it to support multiple Todo Lists. Please note that a full implementation of this extension requires changes to the reducers, components, actions etc. that aren't directly relevant to the topics discussed and have been omitted for brevity.

So far we have only seen selectors receive the Redux store state as an argument, but a selector can receive props too.

Here is an `App` component that renders three `VisibleTodoList` components, each of which has a `listId` prop:

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

Each `VisibleTodoList` container should select a different slice of the state depending on the value of the `listId` prop, so let's modify `getVisibilityFilter` and `getTodos` to accept a props argument:

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

`props` can be passed to `getVisibleTodos` from `mapStateToProps`:

```js
const mapStateToProps = (state, props) => {
  return {
    todos: getVisibleTodos(state, props)
  }
}
```

So now `getVisibleTodos` has access to `props`, and everything seems to be working fine.

**But there is a problem!**

Using the `getVisibleTodos` selector with multiple instances of the `visibleTodoList` container will not correctly memoize:

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

A selector created with `createSelector` only returns the cached value when its set of arguments is the same as its previous set of arguments. If we alternate between rendering `<VisibleTodoList listId="1" />` and `<VisibleTodoList listId="2" />`, the shared selector will alternate between receiving `{listId: 1}` and `{listId: 2}` as its `props` argument. This will cause the arguments to be different on each call, so the selector will always recompute instead of returning the cached value. We'll see how to overcome this limitation in the next section.

### Sharing Selectors Across Multiple Components

> The examples in this section require React Redux v4.3.0 or greater

In order to share a selector across multiple `VisibleTodoList` components **and** retain memoization, each instance of the component needs its own private copy of the selector.

Let's create a function named `makeGetVisibleTodos` that returns a new copy of the `getVisibleTodos` selector each time it is called:

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

We also need a way to give each instance of a container access to its own private selector. The `mapStateToProps` argument of `connect` can help with this.

**If the `mapStateToProps` argument supplied to `connect` returns a function instead of an object, it will be used to create an individual `mapStateToProps` function for each instance of the container.**

In the example below `makeMapStateToProps` creates a new `getVisibleTodos` selector, and returns a `mapStateToProps` function that has exclusive access to the new selector:

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

If we pass `makeMapStateToProps` to `connect`, each instance of the `VisibleTodosList` container will get its own `mapStateToProps` function with a private `getVisibleTodos` selector. Memoization will now work correctly regardless of the render order of the `VisibleTodoList` containers.

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

## Next Steps

Check out the [official documentation](https://github.com/reactjs/reselect) of Reselect as well as its [FAQ](https://github.com/reactjs/reselect#faq). Most Redux projects start using Reselect when they have performance problems because of too many derived computations and wasted re-renders, so make sure you are familiar with it before you build something big. It can also be useful to study [its source code](https://github.com/reactjs/reselect/blob/master/src/index.js) so you don't think it's magic.
