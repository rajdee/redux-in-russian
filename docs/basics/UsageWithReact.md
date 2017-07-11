# Использование с React (Usage with React)

Для начала стоит подчеркнуть, что Redux не имеет отношения к React. Вы можете создавать Redux-приложения c помощью React, Angular, Ember, jQuery или обычного JavaScript.

И все-таки, Redux работает особенно хорошо с такими фреймворками, как [React](http://facebook.github.io/react/) и [Deku](https://github.com/dekujs/deku), потому что они позволяют вам описать UI как функцию состояния и, кроме того, Redux умеет менять состояние (state) приложения в ответ на произошедшие действия (actions).

Для построения нашего простенького todo-приложения мы будем использовать React.

## Установка React Redux

[React bindings](https://github.com/reactjs/react-redux) не включены в redux по умолчанию. Вам нужно установить их явно:

```
npm install --save react-redux
```

Если вы не используете npm, то можете взять последнюю UMD-сборку из unpkg ([development](https://unpkg.com/react-redux@latest/dist/react-redux.js) или [production](https://unpkg.com/react-redux@latest/dist/react-redux.min.js)). Добавив UMD-сборку на страницу при помощи тега `<script>`, вы получите глобальный `window.ReactRedux`.

## Презентационные компоненты и компоненты-контейнеры (Presentational and Container Components)

React-биндинги для Redux руководствуются идеей **разделения компонентов на представления и контейнеры**. Если вы не знакомы с этими терминами, то [сперва прочитайте про них](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), а затем возвращайтесь обратно. Они очень важны, так что мы подождем!

Закончили читать? Давайте перечислим их отличия:

<table>
    <thead>
      <tr>
          <th></th>
          <th scope="col" style="text-align:left" width="36%">Компоненты-представления</th>
          <th scope="col" style="text-align:left" width="41%">Компоненты-контейнеры</th>
      </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">Назначение</th>
          <td>Как выглядит (разметка, стили)</td>
          <td>Как работает (загрузка данных, обновление состояния)</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Знают о Redux</th>
          <td>Нет</th>
          <td>Да</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Читают данные</th>
          <td>Читают данные из props</td>
          <td>Подписываются на Redux-состояние</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Изменяют данные</th>
          <td>Вызывают колбеки из props</td>
          <td>Отправляют Redux-действия</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Написаны</th>
          <td>Руками</td>
          <td>Обычно генерируются React Redux</td>
        </tr>
    </tbody>
</table>

Большинство компонентов, которые мы напишем, будут представлениями, но чтобы соединить их с Redux-состоянием, нам потребуется сгенерировать несколько контейнеров. Это и дальнейшее описание не означает, что компоненты контейнера должны быть расположены ближе к вершине дерева компонентов. Если компонент-контейнер становится слишком сложным, т.е. он имеет сильную вложенность презентационных компонентов, с бесчисленным количеством обратных вызовов, передающихся вниз, используйте еще один контейнер в дереве компонентов, как отмечено в [FAQ](../faq/ReactRedux.md#react-multiple-components).

Технически, вы можете написать контейнеры вручную, используя [`store.subscribe()`](../api/Store.md#subscribe). Мы не советуем вам это делать, потому что React Redux производит много оптимизаций производительности, которые было бы трудно написать руками. По этой причине, вместо того чтобы писать контейнеры, мы генерируем их, воспользовавшись функцией [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options), предоставленной React Redux, об этом ниже.

## Проектирование иерархии компонентов

Помните, как мы [спроектировали структуру корневого объекта состояния](Reducers.md)? В этот раз мы спроектируем иерархию UI-компонентов, которая будет соответствовать этой структуре. С такого рода задачей Вы можете столкнуться, разрабатывая и не Redux-приложение. [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) — великолепное руководство, которое поясняет весь процесс решения этой задачи.

Наш бриф довольно прост. Мы хотим показать список дел (todo). По клику мы должны зачеркнуть дело, что будет означать, что оно выполнено. Также мы хотим показать поле ввода, с помощью которого пользователь сможет добавить новое дело в список. В футере должны быть переключатели, с помощью которых мы будем показывать все дела, только завершенные, только не завершенные.

### Компоненты-представления

Из этого брифа получаются следующие представления и их props:

* **`TodoList`** — список, показывающий видимые todos.
  - `todos: Array` — массив todo-объектов, имеющих форму `{ id, text, completed }`.
  - `onTodoClick(id: number)` — колбек, который будет вызван при клике на todo.
* **`Todo`** — отдельный todo.
  - `text: string` — текст для отображения.
  - `completed: boolean` — должен ли todo показываться зачеркнутым.
  - `onClick()` — колбек, который будет вызван при клике на todo.
* **`Link`** — ссылка с колбеком.
  - `onClick()` — колбек, который будет вызван при клике на ссылку.
* **`Footer`** — область, где мы позволим пользователю менять текущую видимость todos.
* **`App`** — корневой компонент, который рендерит все остальное.

Они описывают *вид*, но не знают *откуда* приходят данные или *как* изменить их. Они только рендерят то, что им дают. Если вы мигрируете с Redux на что-нибудь другое, вы сможете оставить эти компоненты точно такими же. Они не зависят от Redux.

### Компоненты-контейнеры

Нам также потребуются некоторые контейнеры, чтобы соединить представления с Redux. Например, представлению `TodoList` требуется контейнер `VisibleTodoList`, который подписывается на Redux-хранилище и знает, как применять текущий фильтр видимости. Чтобы изменить фильтр видимости, мы предоставим представлению `FilterLink`, контейнер, который рендерит `Link`, а тот, в свою очередь, отправляет (dispatches) соответствующее действие при клике:

* **`VisibleTodoList`** — фильтрует todos согласно текущему фильтру видимости и рендерит `TodoList`.
* **`FilterLink`** — получает текущий фильтр видимости и рендерит `Link`.
  - `filter: string` — текущий фильтр видимости.

### Другие компоненты

Иногда трудно сказать, каким должен быть компонент — представлением или контейнером. Например, иногда форма и функция действительно соединены вместе, как в случае с этим миниатюрным компонентом:

* **`AddTodo`** — инпут с кнопкой "Добавить"

Технически мы могли бы "разбить" его на два компонента, но это может быть слишком рано на данном этапе. Вполне допустимо смешивать представление и логику, когда компонент очень маленький. Как только он вырастет, станет более понятно как "разбить" его, так что мы пока оставим его смешанным.

## Реализуем компоненты

Давайте напишем компоненты! Мы начнем с представлений, так что пока нам не нужно думать о привязке к Redux. 

### Компоненты-представления

Это все обычные React-компоненты, поэтому мы не будем изучать их детально. Мы пишем функциональные stateless-компоненты, пока нам не потребуются локальное состояние или lifecycle-методы. Это не значит, что представления *должны быть* функциями, просто так легче. Если/когда вам потребуется добавить локальное состояние, lifecycle-методы или оптимизацию производительности, вы сможете конвертировать их в классы.

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

### Компоненты-контейнеры

А теперь настало время подключить эти компоненты представления к Redux, создав некоторые компоненты-контейнеры. Технически, контейнер — это просто React-компонент, который использует [`store.subscribe()`](../api/Store.md#subscribe) для чтения части Redux-дерева состояний и поставляет props представлению, которое он рендерит.
Вы можете написать компонент-контейнер вручную, но вместо этого мы предлагаем генерировать контейнеры с помощью библиотечной функции React Redux [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options), которая предоставляет много полезных оптимизаций для предотвращения ненужных ре-рендеров. (Одним из результатов этого является то, что вам больше не придется беспокоиться о [React performance suggestion](https://facebook.github.io/react/docs/advanced-performance.html) своей реализации `shouldComponentUpdate`).

Чтобы использовать `connect()`, вам нужно определить специальную функцию `mapStateToProps`, которая говорит, как трансформировать текущее Redux-состояние хранилища в props, которые вы хотите передать в оборачиваемое (контейнером) представление. Например, `VisibleTodoList` требуется вычислить `todos` для передачи в `TodoList`, так что нам нужно определить функцию, которая фильтрует `state.todos` согласно `state.visibilityFilter`, и использовать ее в `mapStateToProps`:

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

В дополнение к чтению состояния контейнеры могут отправлять действия (dispatch actions). В похожем стиле вы можете определить функцию `mapDispatchToProps()`, которая получает метод [`dispatch()`](../api/Store.md#dispatch) и возвращает колбек props, который вы можете вставить в представление. Например, мы хотим, чтобы контейнер `VisibleTodoList` вставил prop `onTodoClick` в представление `TodoList` и еще мы хотим, чтобы `onTodoClick` отправлял `TOGGLE_TODO` действие:

```js
const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}
```

Наконец, вызовом `connect()` мы создаем `VisibleTodoList` и передаем эти две функции:

```js
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

Это основы React Redux API, но там есть несколько комбинаций и мощных опций, поэтому мы рекомендуем вам подробно изучить [эту документацию](https://github.com/reactjs/react-redux). В случае если вы переживаете, что `mapStateToProps` создает слишком много новых объектов, то вам будет полезно узнать о [вычислении полученных данных](../recipes/ComputingDerivedData.md) и [reselect](https://github.com/reactjs/reselect).

Остальные компоненты контейнеры вы найдете ниже:

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

### Другие Компоненты

#### `containers/AddTodo.js`

```js
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form onSubmit={e => {
        e.preventDefault()
        if (!input.value.trim()) {
          return
        }
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        <input ref={node => {
          input = node
        }} />
        <button type="submit">
          Add Todo
        </button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

## Передаем Хранилище

Всем компонентам-контейнерам необходим доступ к Redux-хранилищу (store), для того чтобы они могли подписаться на него. Как вариант — передать его как prop в каждый контейнер. Однако это становится утомительным, так как вы должны подключать `store`, даже если представления просто рендерят контейнер глубоко в дереве компонентов.

Мы рекомендуем другой вариант — использовать специальный React Redux компонент [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store), вызов которого [магически](https://facebook.github.io/react/docs/context.html) делает хранилище доступным всем контейнерам в приложении без его явной передачи. Вам нужно только воспользоваться им единожды, когда вы рендерите корневой компонент:

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

Прочитайте [полный исходный код для этого руководства](ExampleTodoList.md) для лучшего усваивания полученных знаний. А затем прямиком в [руководство для опытных](../advanced/README.md) для изучения обработки сетевых запросов и роутинга!
