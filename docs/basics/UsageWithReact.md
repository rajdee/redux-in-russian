# Использование в связке с React (Usage with React)

Для начала стоит подчеркнуть, что Redux не имеет отношения к React. Вы можете создавать Redux приложения c помощью React, Angular, Ember, jQuery или обычного javascript.

И все-таки, Redux работает особенно хорошо с такими фреймворками, как [React](http://facebook.github.io/react/) и [Deku](https://github.com/dekujs/deku) потому что они позволяют Вам описать UI, как функцию состояния, и, кроме того, Redux умеет менять состояние (state) приложения в ответ на произошедшие действия (actions).

Для построения нашего простенького ToDo приложения мы будем использовать React.

## Установка React Redux (Installing React Redux)

[React bindings](https://github.com/gaearon/react-redux) не включены в redux по умолчанию. Вам нужно установить их явно:

```
npm install --save react-redux
```

## Компоненты-контейнеры и презентационные компонены (Container and Presentational Components)

React bindings для Redux охвачены идеей [разделения компонентов на компоненты-контейнеры и презентационные компонеты](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

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
          <td>Вызывают каллбэки из props</td>
        </tr>
    </tbody>
</table>

В нашем ToDo приложении будет один компонент-контейнер на вершине иерархии представлений (view). В более сложных приложениях таких "умных" компонентов может быть несколько. Мы советуем не вкладывать "умные" компоненты друг в друга, а передавать `props` вниз по иерархии компонентов всегда, когда это возможно.

## Проектирование иерархии компонентов (Designing Component Hierarchy)

Помните как мы [спроектировали структуру объекта состояния](Reducers.md)? В этот раз мы спроектируем иерархию UI компонентов, которая будет соответствовать этой структуре. С такого рода задачей Вы можете столкнуться разрабатывая и не Redux приложение. [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) - великолепное руководство, которое поясняет весь процесс решения этой задачи.

Наше техническое задание для проектирования довольно простое. Мы хотим показать список дел. По клику мы должны зачеркнуть дело, что будет означать, что оно выполнено. Также мы хотим показать поле ввода, с помощью которого пользователь сможет добавить новое дело в список. В футере должны быть переключатели, с помощью которых мы будем показывать все дела / только завершенные / только не завершенные.

Следуя спецификации, можно выделить такие компоненты и их свойства:

* **`AddTodo`** поле ввода с кнопкой.
    * `onAddClick(text: string)` функция-колбек, которая будет вызвана при нажатии на кнопку.
* **`TodoList`** список, в котором отображены видимые дела.
    * `todos: Array` массив объектов дел, имеющих следующую структуру `{ text, completed }`.
    * `onTodoClick(index: number)` функция-колбек, которая будет вызвана по клику на дело.
* **`Todo`** единичный элемент-дело.
    * `text: string` текст для отображения.
    * `completed: boolean` должно ли дело быть показано отмеченным (вычеркнутым).
    * `onClick()` функция-колбек, которая будет вызвана по клику на дело.
* **`Footer`** компонент, в котором мы дадим юзеру возможность изменять фильтрацию списка.
    * `filter: string` текущий фильтр: `'SHOW_ALL'`, `'SHOW_COMPLETED'` или `'SHOW_ACTIVE'`.
    * `onFilterChange(nextFilter: string)`: функция-колбек, которая будет вызвана если пользователь выберет другой фильтр.

Все это - презентационные компоненты. Они не знают **откуда** приходят данные и **как** их изменить. Они лишь только рендерят то, что им передали.

Если Вы замените Redux чем-то другим, то все равно сможете пользоваться этими компонентами. Они никак не зависят от Redux.

Давайте напишем их! Пока нам не нужно думать о привязке Redux. Мы можем просто передавать им фейковые данные, чтобы убедиться, что компоненты рендерятся корректно.

## Презентационные компоненты (Presentational Components)

Это обычные React компоненты, так что давайте не будем сильно задерживаться на их изучении. Вот они:

#### `components/AddTodo.js`

```js
import React, { Component, PropTypes } from 'react'

export default class AddTodo extends Component {
  render() {
    return (
      <div>
        <input type='text' ref='input' />
        <button onClick={e => this.handleClick(e)}>
          Add
        </button>
      </div>
    )
  }

  handleClick(e) {
    const node = this.refs.input
    const text = node.value.trim()
    this.props.onAddClick(text)
    node.value = ''
  }
}

AddTodo.propTypes = {
  onAddClick: PropTypes.func.isRequired
}
```

#### `components/Todo.js`

```js
import React, { Component, PropTypes } from 'react'

export default class Todo extends Component {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none',
          cursor: this.props.completed ? 'default' : 'pointer'
        }}>
        {this.props.text}
      </li>
    )
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
}
```

#### `components/TodoList.js`

```js
import React, { Component, PropTypes } from 'react'
import Todo from './Todo'

export default class TodoList extends Component {
  render() {
    return (
      <ul>
        {this.props.todos.map((todo, index) =>
          <Todo {...todo}
                key={index}
                onClick={() => this.props.onTodoClick(index)} />
        )}
      </ul>
    )
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
}
```

#### `components/Footer.js`

```js
import React, { Component, PropTypes } from 'react'

export default class Footer extends Component {
  renderFilter(filter, name) {
    if (filter === this.props.filter) {
      return name
    }

    return (
      <a href='#' onClick={e => {
        e.preventDefault()
        this.props.onFilterChange(filter)
      }}>
        {name}
      </a>
    )
  }

  render() {
    return (
      <p>
        Show:
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    )
  }
}

Footer.propTypes = {
  onFilterChange: PropTypes.func.isRequired,
  filter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}
```

Вот и все! Мы можем проверить работоспособность наших компонентов, написав простенький `App`, который будет рендерить их:

#### `containers/App.js`

```js
import React, { Component } from 'react'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

export default class App extends Component {
  render() {
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            console.log('add todo', text)
          } />
        <TodoList
          todos={
            [
              {
                text: 'Use Redux',
                completed: true
              },
              {
                text: 'Learn to connect it to React',
                completed: false
              }
            ]
          }
          onTodoClick={index =>
            console.log('todo clicked', index)
          } />
        <Footer
          filter='SHOW_ALL'
          onFilterChange={filter =>
            console.log('filter change', filter)
          } />
      </div>
    )
  }
}
```

Вот то, что мы увидим, когда отрендерим `<App />`:

<img src='http://i.imgur.com/lj4QTfD.png' width='40%'>

Само по себе это не очень интересно. Давайте прикрутим Redux!

## Интеграция Redux (Connecting to Redux)

Мы должны сделать два изменения для того, чтобы интегрировать Redux в наш `App` и научить его запускать действия (dispatch actions) и читать состояние (state) из Redux хранилища (store).

Для начала нам нужно импортировать `Provider` из [`react-redux`](http://github.com/gaearon/react-redux), который мы установили чуть раньше, и **обернуть корневой компонент в `<Provider>`** перед рендерингом.

#### `index.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './containers/App'
import todoApp from './reducers'

let store = createStore(todoApp)

let rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

Это сделает наш экземпляр хранилища доступным для всех нижестоящих компонентов. (Внутри это реализовано благодаря [возможности React - “context”](http://facebook.github.io/react/docs/context.html))

Затем нам **нужно обернуть компоненты, которые мы хотим связать с Redux, в вызов функции `connect()` из [`react-redux`](http://github.com/gaearon/react-redux)** Старайтесь делать это только с компонентами верхнего уровня или обработчиками роутов. Хотя, технически, Вы можете обернуть в вызов `connect()` любой компонент, старйтесь избегать этого на более глубоких уровнях иерархии компонентов, т.к. это усложнит отслеживание потока данных.

**Любой компонент, обернутый в вызов функции `connect()`, получит функцию [`dispatch`](../api/Store.md#dispatch), как свойство (as a prop) и любое состояние, которое ему потребуется, из глобального состояния**. Функция `connect()` имеет единственный аргумент (единственный в контексте данного примера, а [вообще их 4](https://github.com/rackt/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) *прим. переводчика*) - функцию, которую мы назовем **селектор (selector)**. Эта функция принимает глобальное состояние хранилища Redux и должна возвращать props, которые нужны компоненту. В простейшем случае, Вы просто можете вернуть из этой функции приходящий в нее глобальный `state`, но, скорее всего, Вы захотите сначала немного его преобразовать. (Т.е. благодаря этой функции мы можем из всего огромного дерева состояния, которое хранится в Redux хранилище, вытащить именно те ветки/срезы/куски состояния, которые нужны конкретному компоненту *прим. переводчика*)

Для того, чтобы делать высококачественные, оптимизированные в плане производительности и памяти (мемоизацией например) трансформации состояния, с помощью компонуемых селекторов (речь идет о функции *selector*, описанной выше), Вы должны обратить внимние на [reselect](https://github.com/faassen/reselect). В этом примере мы не будем его использовать, но он отлично работает в больших приложениях. 

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

class App extends Component {
  render() {
    // Получено благодаря вызову connect():
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    )
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired,
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}

function selectTodos(todos, filter) {
  switch (filter) {
    case VisibilityFilters.SHOW_ALL:
      return todos
    case VisibilityFilters.SHOW_COMPLETED:
      return todos.filter(todo => todo.completed)
    case VisibilityFilters.SHOW_ACTIVE:
      return todos.filter(todo => !todo.completed)
  }
}

// Какие именно props мы хотим получить из приходящего, как аргумент глобального состояния?
// Обратите внимание: используйте https://github.com/faassen/reselect для лучшей производительности.
function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  }
}

// Оборачиваем компонент `App` для внедрения в него функции `dispatch` и состояния
export default connect(select)(App)
```

Вот и все! Простенькое ToDo приложение теперь полностью работоспособно.

## Следующие шаги

Прочитайте [полный исходный код для этого руководства](ExampleTodoList.md) для лучшего усваивания полученных знаний. А затем, прямиком в [руководство для опытных](../advanced/README.md) для изучения обработки сетевых запросов и роутинга!
