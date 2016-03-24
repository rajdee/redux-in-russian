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
Важно понимать эту концепцию.

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
Большинство компонентов, которые мы напишем, будут презентационными. Но нам нужно сгенерировать несколько компонентов-контейнеров, чтобы соединить их с хранилищем Redux.
 
Технически Вы можете написать компоненты-контейнеры вручную, используя [`store.subscribe()`](../api/Store.md#subscribe). Но это не рекомендуется делать , т.к в React Redux реализовано много оптимизаций производительности. Поэтому, вместо того чтобы писать самим, мы будет генерировать компоненты-контейнеры с помощью [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options). Далее Вы увидите как мы будем это делать.


## Проектирование иерархии компонентов (Designing Component Hierarchy)

Помните как мы [спроектировали структуру объекта состояния](Reducers.md)? В этот раз мы спроектируем иерархию UI компонентов, которая будет соответствовать этой структуре. С такого рода задачей Вы можете столкнуться разрабатывая и не Redux приложение. [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) - великолепное руководство, которое поясняет весь процесс решения этой задачи.

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

Иногда трудно понять будет компонент "презентационным" или "контейнером". Например, иногда форма ввода и функция сильно связаны, как в нашем случае маленький компонент `AddTodo`:

* **`AddTodo`** Поле ввода с кнопкой "Добавить" (Add)

Технически мы можем разделить это на два компонента, но это может быть слишком рано на этом этапе. Это нормально смешивать представление и логику в компоненте, который очень мал. По мере роста приложения, станет вполне очевидно, когда нужно будет разделить на компоненты. В нашем случае мы делить не будем.

## Имплементация компонентов

Давайте напишем компоненты! Начнем с презентационных компонентов, пока нам не нужно думать о связке с Redux.

### Презентационные Компоненты (Presentational Components)

Это обычные React компоненты, ничего особенного. Мы пишем функциональные stateless компоненты, пока нам не будут нужны локальный state и методы lifecycle. Это не значит что презентационные компоненты *должны быть* функциональными, так просто легче их объявлять. Когда вам понадобятся  локальный state, lifecycle методы, оптимизация - вы можете конвертировать компоненты в классы.

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

###  Компоненты Контейнеры (Container Components)

Настало время связать наши презентационные компоненты с Redux с помощью создания нескольких компонентов-контейнеров. Технически компонент-контейнер это просто React компонент, который использует [`store.subscribe()`](../api/Store.md#subscribe) для чтения части дерева состояния (state tree) Redux, а также предоставляет props  презентационным компонентам, которые их рендерят. Вы можете написать контейнер сами, но React Redux включает много полезных оптимизаций, потому мы советуем генерировать контейнер с помощью  [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) функцией из библиотеки React Redux

Чтобы использовать `connect()`, вам нужно определить специальную функцию `mapStateToProps`, которая скажет как трансформировать текущее состояние хранилища (store state) Redux в props'ы , которые вы хотите передать в оборачиваемые презентационные компоненты.
Например, `VisibleTodoList` будет определять передаваемые `todos` в `TodoList`, поэтому мы напишем функцию, которая будет фильтровать `state.todos` согласно `state.visibilityFilter`, и будет использовать это в своём `mapStateToProps`.



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
В добавление к чтению состояния (state), компонент-контейнер может отправлять действия (dispatch actions). В том же духе, вы можете определить функию `mapDispatchToProps()`, которая принимает [`dispatch()`](../api/Store.md#dispatch) метод и возвращает колбэк props, которые вы хотите вставить в презентационный компонент. Например, мы хотим чтобы `VisibleTodoList` вставил prop `onTodoClick` в `TodoList` компонент, еще мы хотим `onTodoClick` чтобы отправить (dispatch) `TOGGLE_TODO` действие (action).


```js
const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}
```
В окончании, мы создадим `VisibleTodoList` через вызов `connect()` и передадим наши две функции.

```js
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```
Это базовая часть API React Redux, так же мы рекомендуем посмотреть остальную часть [its documentation](https://github.com/reactjs/react-redux) детально.

In case you are worried about `mapStateToProps` creating new objects too often, you might want to learn about [computing derived data](../recipes/ComputingDerivedData.md) with [reselect](https://github.com/rackt/reselect).

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

## Передача хранилища (Passing the Store)

Всем компонентам контейнерам необходим доступ к хранилищу Redux, поэтому они могут подписаться на него. Одним из вариантов было бы передать его в качестве prop для каждого контейнера компонента.
 Но это довольно утомительно - "пробрасывать"  по дереву компонентов `store`, даже через презентационные компоненты  - из-за того что они содержат в себе компоненты-контейнеры

Для этого мы рекомендуем использвать специальный React Redux компонент [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) чтобы  [магическим образом](https://facebook.github.io/react/docs/context.html) делать доступным хранилище всем компонентам контейнерам в приложение без явной передачи. Вам нужно использовать его только однажды, в рендере root компонента


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




