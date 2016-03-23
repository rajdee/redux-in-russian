# Использование в связке с React (Usage with React)

Для начала стоит подчеркнуть, что Redux не имеет отношения к React. Вы можете создавать Redux приложения c помощью React, Angular, Ember, jQuery или обычного javascript.

И все-таки, Redux работает особенно хорошо с такими фреймворками, как [React](http://facebook.github.io/react/) и [Deku](https://github.com/dekujs/deku) потому что они позволяют Вам описать UI как функцию состояния, и, кроме того, Redux умеет менять состояние (state) приложения в ответ на произошедшие действия (actions).

Для построения нашего простенького ToDo приложения мы будем использовать React.

## Установка React Redux (Installing React Redux)

[React bindings](https://github.com/gaearon/react-redux) не включены в redux по умолчанию. Вам нужно установить их явно:

```
npm install --save react-redux
```

Если вы не используете npm, Вы можете взять последнюю  версию UMD сборки (build) на сервисе npmcdn (либо [development](https://npmcdn.com/react-redux@latest/dist/react-redux.js) либо [production](https://npmcdn.com/react-redux@latest/dist/react-redux.min.js) ). The UMD build exports a global called `window.ReactRedux` if you add it to your page via a `<script>` tag.


## Компоненты-контейнеры и презентационные компоненты (Container and Presentational Components)

React bindings для Redux охвачены идеей [разделения компонентов на компоненты-контейнеры и презентационные компоненты](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

Желательно, чтобы только компоненты верхнего уровня вашего приложения (например, обработчики роутов (route handlers) знали о Redux. Компоненты, которые находятся ниже в иерархии, должны быть презентационными и принимать все данные только через `props`.

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Компоненты- контейнеры</th>
            <th scope="col" style="text-align:left">Презентационные Компоненты</th>
        </tr>
    </thead>
    <tbody>
          <tr>
          <th scope="row" style="text-align:right">Назначение</th>
          <td>Отвечают за логику ( работа с данными, обновление состояния (state) )</td>
          <td>Отвечают за внешний вид ( разметка, стили ) </td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Расположение</th>
          <td>Верхний уровень, обработчики роутов</td>
          <td>Средний уровень и компоненты-"листья" (leaf components)</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Знают о Redux</th>
          <td>Да</th>
          <td>Нет</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Получают / читают данные</th>
          <td>Подписываются на Redux состояние (Redux state)</td>
          <td>Получают данные из props</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Изменяют данные</th>
          <td>Отправляют Redux действия (actions)</td>
          <td>Вызывают колбэки из props</td>
        </tr>
          <tr>
          <th scope="row" style="text-align:right">Пишутся</th>
          <td>Обычно генерируются React Redux</td>
          <td>Руками</td>
        </tr>
    </tbody>
</table>
Большинство компонентов, которые мы напишем, будут презентационными. Но нам нужно сгенерировать несколько компонентов-контейнеров, чтобы соеденить их с хранилищем Redux.
 
Технически Вы можете написать компоненты-контейнеры вручную, используя [`store.subscribe()`](../api/Store.md#subscribe). Но это не рекомендуется делалать , т.к в React Redux релизовано много оптимизаций производительности. Поэтому, вместо того чтобы писать самим, мы будет генерировать компоненты-контейнеры с помощью [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options). Далее Вы увидете как мы будем это делать.


## Проектирование иерархии компонентов (Designing Component Hierarchy)

Помните как мы [спроектировали структуру объекта состояния](Reducers.md)? В этот раз мы спроектируем иерархию UI компонентов, которая будет соответствовать этой структуре. С такого рода задачей Вы можете столкнуться разрабатывая и не Redux приложение. [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) - великолепное руководство, которое поясняет весь процесс решения этой задачи.

Our design brief is simple. We want to show a list of todo items. On click, a todo item is crossed out as completed. We want to show a field where the user may add a new todo. In the footer, we want to show a toggle to show all, only completed, or only active todos.

Наше техническое задание для проектирования довольно простое. Мы хотим показать список дел. По клику мы должны зачеркнуть дело, что будет означать, что оно выполнено. Также мы хотим показать поле ввода, с помощью которого пользователь сможет добавить новое дело в список. В футере должны быть переключатели, с помощью которых мы будем показывать все дела / только завершенные / только не завершенные.

### Презентационные Компоненты (Presentational Components)

Следуя спецификации, можно выделить такие компоненты и их свойства (props):

* **`TodoList`** список, в котором отображены видимые дела.
    - `todos: Array` массив объектов дел, имеющих следующую структуру `{ id, text, completed }`.
    - `onTodoClick(id: number)` функция-колбек, которая будет вызвана по клику на дело.
* **`Todo`** единичный элемент-дело.
    - `text: string` текст для отображения.
    - `completed: boolean` должно ли дело быть показано отмеченным (вычеркнутым).
    - `onClick()` функция-колбек, которая будет вызвана по клику на дело.
* **`Link`** Ссылка с функцией-колбеком.
  - `onClick()` функция-колбек, которая будет вызвана по клику на ссылку.
* **`Footer`** компонент, в котором мы дадим юзеру возможность изменять фильтрацию списка.
* **`App`** Это корневой компонент, который будет рендерить всё остальное.

Все это - презентационные компоненты. Они описывают внешний вид, но не знают **откуда** приходят данные и **как** их изменить. Они лишь только рендерят то, что им передали.

Если Вы замените Redux чем-то другим, то все равно сможете пользоваться этими компонентами. Они никак не зависят от Redux.

## Компоненты Контейнеры (Container Components)

Нам также нужны Компоненты Контейнеры, чтобы связать презентационные компоненты с Redux. Например, презентационный компонент `TodoList` нуждается в компоненте `VisibleTodoList`, который подписан на хранилище (store) Redux и знает как применить текущее состояние фильтрации (visibility filter). Чтобы менять состояние фильтрации (visibility filter), мы создадим компонент-контейнер `FilterLink`, который рендерит презентационный компонент `Link`. `Link` будет отправлять подходящее действие (action) по клику:

* **`VisibleTodoList`** Фильтрует список дел согласно текущему состоянию фильтрации(visibility filter) и рендерит `TodoList`.
* **`FilterLink`** получает текущее состояние фильтрации(visibility filter) и рендерит `Link`.
  - `filter: string` состояние фильтрации.


### Другие компоненты (Other Components)

Sometimes it’s hard to tell if some component should be a presentational component or a container. For example, sometimes form and function are really coupled together, such as in case of this tiny component:

* **`AddTodo`** is an input field with an “Add” button

Technically we could split it into two components but it might be too early at this stage. It’s fine to mix presentation and logic in a component that is very small. As it grows, it will be more obvious how to split it, so we’ll leave it mixed.

## Implementing Components

Let’s write the components! We begin with the presentational components so we don’t need to think about binding to Redux yet.

### Presentational Components

These are all normal React components, so we won’t examine them in detail. We write functional stateless components unless we need to use local state or the lifecycle methods. This doesn’t mean that presentational components *have to* be functions—it’s just easier to define them this way. If and when you need to add local state, lifecycle methods, or performance optimizations, you can convert them to classes.

#### `components/Todo.js`

```js
import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

#### `components/TodoList.js`

```js
import React, { PropTypes } from 'react'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

#### `components/Link.js`

```js
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

#### `components/Footer.js`

```js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default Footer
```

#### `components/App.js`

```js
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
)

export default App
```

### Container Components

Now it’s time to hook up those presentational components to Redux by creating some containers. Technically, a container component is just a React component that uses [`store.subscribe()`](../api/Store.md#subscribe) to read a part of the Redux state tree and supply props to a presentational component it renders. You could write a container component by hand but React Redux includes many useful optimizations so we suggest to generate container components with [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) function from the React Redux library.

To use `connect()`, you need to define a special function called `mapStateToProps` that tells how to transform the current Redux store state into the props you want to pass to a presentational component you are wrapping. For example, `VisibleTodoList` needs to calculate `todos` to pass to the `TodoList`, so we define a function that filters the `state.todos` according to the `state.visibilityFilter`, and use it in its `mapStateToProps`:

```js
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
```

In addition to reading the state, container components can dispatch actions. In a similar fashion, you can define a function called `mapDispatchToProps()` that receives the [`dispatch()`](../api/Store.md#dispatch) method and returns callback props that you want to inject into the presentational component. For example, we want the `VisibleTodoList` to inject a prop called `onTodoClick` into the `TodoList` component, and we want `onTodoClick` to dispatch a `TOGGLE_TODO` action:

```js
const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}
```

Finally, we create the `VisibleTodoList` by calling `connect()` and passing these two functions:

```js
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

These are the basics of the React Redux API, but there are a few shortcuts and power options so we encourage you to check out [its documentation](https://github.com/reactjs/react-redux) in detail. In case you are worried about `mapStateToProps` creating new objects too often, you might want to learn about [computing derived data](../recipes/ComputingDerivedData.md) with [reselect](https://github.com/rackt/reselect).

Find the rest of the container components defined below:

#### `containers/FilterLink.js`

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

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

### Other Components

#### `containers/AddTodo.js`

```js
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <input ref={node => {
        input = node
      }} />
      <button onClick={() => {
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        Add Todo
      </button>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

## Passing the Store

All container components need access to the Redux store so they can subscribe to it. One option would be to pass it as a prop to every container component. However it gets tedious, as you have to wire `store` even through presentational components just because they happen to render a container deep in the component tree.

The option we recommend is to use a special React Redux component called [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) to [magically](https://facebook.github.io/react/docs/context.html) make the store available to all container components in the application without passing it explicitly. You only need to use it once when you render the root component:

#### `index.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```


## Следующие шаги

Прочитайте [полный исходный код для этого руководства](ExampleTodoList.md) для лучшего усваивания полученных знаний. А затем, прямиком в [руководство для опытных](../advanced/README.md) для изучения обработки сетевых запросов и роутинга!


////////////////////////////////////////////////////////////////////////
Вот и все! Мы можем проверить работоспособность наших компонентов, написав простенький `App`, который будет рендерить их:



Вот то, что мы увидим, когда отрендерим `<App />`:

<img src='http://i.imgur.com/lj4QTfD.png' width='40%'>

Само по себе это не очень интересно. Давайте прикрутим Redux!

## Интеграция Redux (Connecting to Redux)

Мы должны сделать два изменения для того, чтобы интегрировать Redux в наш `App` и научить его запускать действия (dispatch actions) и читать состояние (state) из Redux хранилища (store).

Для начала нам нужно импортировать `Provider` из [`react-redux`](http://github.com/gaearon/react-redux), который мы установили чуть раньше, и **обернуть корневой компонент в `<Provider>`** перед рендерингом.


Это сделает наш экземпляр хранилища доступным для всех нижестоящих компонентов. (Внутри это реализовано благодаря [возможности React - “context”](http://facebook.github.io/react/docs/context.html))

Затем нам **нужно обернуть компоненты, которые мы хотим связать с Redux, в вызов функции `connect()` из [`react-redux`](http://github.com/gaearon/react-redux)** Старайтесь делать это только с компонентами верхнего уровня или обработчиками роутов. Хотя, технически, Вы можете обернуть в вызов `connect()` любой компонент, старайтесь избегать этого на более глубоких уровнях иерархии компонентов, т.к. это усложнит отслеживание потока данных.

**Любой компонент, обернутый в вызов функции `connect()`, получит функцию [`dispatch`](../api/Store.md#dispatch), как свойство (as a prop) и любое состояние, которое ему потребуется, из глобального состояния**. Функция `connect()` имеет единственный аргумент (единственный в контексте данного примера, а [вообще их 4](https://github.com/rackt/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) *прим. переводчика*) - функцию, которую мы назовем **селектор (selector)**. Эта функция принимает глобальное состояние хранилища Redux и должна возвращать props, которые нужны компоненту. В простейшем случае, Вы просто можете вернуть из этой функции приходящий в нее глобальный `state`, но, скорее всего, Вы захотите сначала немного его преобразовать. (Т.е. благодаря этой функции мы можем из всего огромного дерева состояния, которое хранится в Redux хранилище, вытащить именно те ветки/срезы/куски состояния, которые нужны конкретному компоненту *прим. переводчика*)

Для того, чтобы делать высококачественные, оптимизированные в плане производительности и памяти (мемоизацией, например) трансформации состояния с помощью компонуемых селекторов (речь идет о функции *selector*, описанной выше), Вы должны обратить внимание на [reselect](https://github.com/faassen/reselect). В этом примере мы не будем его использовать, но он отлично работает в больших приложениях. 



Вот и все! Простенькое ToDo приложение теперь полностью работоспособно.

