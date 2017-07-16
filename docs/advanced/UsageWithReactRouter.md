# Использование с React Router

Итак, вы хотите использовать маршрутизацию с Вашим Redux-приложением. Для этого вы можете использовать [React Router](https://github.com/reactjs/react-router). Тогда Redux будет источником правдивых данных, а React Router — единственным источником URL. В большинстве случаев, ***правильно*** разделять эти понятия, но до тех пор, пока вам не понадобится путешествовать во времени (Time Travel) и перематывать действия, которые изменяют URL.

## Установка React Router
`react-router` доступно в npm . В этом руководстве преполагается использование версии `react-router@^2.7.0`.

`npm install --save react-router`

## Настройка запасного URL

Перед внедрением React Router нам требуется настроить наш сервер разработки. Конечно, наш сервер может не знать о роутах, заявленных в настройках React Router. Например, если вы переходите по ссылке `/todos`, то Ваш сервер должен возвращать `index.html`, так как это одностраничное приложение. Далее идут примеры настройки популярных серверов.

>### Важно при использовании Create React App

> Если вы используете Create React App, то вам не требуется настраивать запасной URL. Это уже сделано автоматически.

### Настройка Express
Если вы получаете `index.html` из Express:
``` js
  app.get('/*', (req,res) => {
    res.sendfile(path.join(__dirname, 'index.html'))
  })
```

### Настройка WebpackDevServer
Если вы получаете `index.html` из WebpackDevServer:
Добавьте в webpack.config.dev.js:
```js
  devServer: {
    historyApiFallback: true,
  }
```

## Подключение React Router к Redux-приложению

В этой главе мы будем использовать наш пример [Todos](https://github.com/reactjs/redux/tree/master/examples/todos). Рекомендуем вам склонировать его для этой главы.

Во-первых, нам надо импортировать `<Router />` и `<Route />` из React Router. Вот как это делается:

```js
import { Router, Route, browserHistory } from 'react-router';
```

В React-приложении обычно `<Route />` оборачивается в `<Router />`, так что когда URL меняется, `<Router />` находит часть своих роутов и рендерит их сформированные компоненты. `<Route />` используется для (неофициального) сопоставления роутов иерархии компонентов вашего приложения. Вы можете объявить в `path` путь, используемый в URL, и в `component` — компонент, который должен быть сгенерирован при совпадении с этим URL.

```js
const Root = () => (
  <Router>
    <Route path="/" component={App} />
  </Router>
);
```

Однако, в нашем Redux-приложении нам все еще требуется `<Provider />` — компонент высшего порядка, поставляемый с React Redux, который позволяет привязать Redux к React (см. [Использование с React](../basics/UsageWithReact.md)).

Далее мы импортируем `<Provider />` из React Redux:

```js
import { Provider } from 'react-redux';
```

Обернем `<Router />` в `<Provider />`, таким образом обработчики маршрутизации смогут получить [доступ к `хранилищу`](../basics/UsageWithReact.html#передаем-хранилище).

```js
const Root = ({ store }) => (
  <Provider store={store}>
    <Router>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
```

Теперь компонент `<App />` будет отрендерен, если URL содержит '/'. Кроме того, мы будем добавлять необязательный `(:filter)` параметр к `/`. Нам это потребуется позже, когда мы будем читать параметр `(:filter)` из URL.

```js
<Route path="/(:filter)" component={App} />
```

Вероятно, Вы хотите убрать хеш из URL (например: `http://localhost:3000/#/?_k=4sbb0i`). Для этого вам потребуется импортировать `browserHistory` из React Router:

```js
import { Router, Route, browserHistory } from 'react-router';
```

и передать её в `<Router />` для удаления хеша из URL:

```js
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
```

Если Вам нужна поддержка таких старых браузеров, как IE9, то Вы также можете использовать `browserHistory`.

#### `components/Root.js`
``` js
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
  </Provider>
);

Root.propTypes = {
  store: PropTypes.object.isRequired,
};

export default Root;
```

## Навигация при помощи React Router

React Router поставляется с компонентом [`<Link />`](https://github.com/reactjs/react-router/blob/master/docs/API.md#link), который позволяет перемещаться по приложению. Мы можем использовать его в нашем компоненте-контейнере `<FilterLink />`, таким образом, мы сможем изменять URL при помощи `<FilterLink />`. Параметр `activeStyle={}` позволяет применить стиль активного состояния.

#### `containers/FilterLink.js`
```js
import React from 'react';
import { Link } from 'react-router';

const FilterLink = ({ filter, children }) => (
  <Link
    to={filter === 'all' ? '' : filter}
    activeStyle={{
      textDecoration: 'none',
      color: 'black'
    }}
  >
    {children}
  </Link>
);

export default FilterLink;
```

#### `containers/Footer.js`
```js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="all">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="active">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="completed">
      Completed
    </FilterLink>
  </p>
);

export default Footer
```

Теперь, если вы кликнете на `<FilterLink />`, вы увидите, что URL изменится с `'/completed'`, `'/active'`, `'/'`. Даже если вы захотите перейти назад, использую соответствующую кнопку браузера, приложение будет использовать для этого историю Вашего браузера и успешно перейдет к предыдущему URL.

## Чтение из URL

Сейчас todo list не фильтруется после изменения URL. Это происходит потому, что фильтрация описана в функции `mapStateToProps()` компонента `<VisibleTodoList />`, которая в свою очередь связана с `состоянием`, а не с URL. `mapStateToProps` имеет второй необязательный аргумент `ownProps` — объект, содержащий все параметры, переданные в `<VisibleTodoList />`.

#### `containers/VisibleTodoList.js`
```js
const mapStateToProps = (state, ownProps) => {
  return {
    todos: getVisibleTodos(state.todos, ownProps.filter) 
    // ранее было getVisibleTodos(state.todos, state.visibilityFilter)
  };
};
```

В данный момент мы ничего не передаем в `<App />`, поэтому `ownProps` является пустым объектом. Чтобы отфильтровать наши todos в соответствии с URL, нам надо передать параметры URL в `<VisibleTodoList />`.

Когда мы написали:  `<Route path="/(:filter)" component={App} />`, это позволило в компоненте `App` получить доступ к параметру `params`.

Параметр `params` — это объект со всеми параметрами из URL. Например, `params` эквивалетно `{ filter: 'completed' }`, если URL `localhost:3000/completed`. Теперь мы можем считывать URL из компонента `<App />`.*

Важно помнить, что мы используем [ES6-деструкцию](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) на параметрах, чтобы передать их в `params` в `<VisibleTodoList />`.

#### `components/App.js`
```js
const App = ({ params }) => {
  return (
    <div>
      <AddTodo />
      <VisibleTodoList
        filter={params.filter || 'all'}
      />
      <Footer />
    </div>
  );
};
```

## Следующие шаги

Теперь, когда мы знаем основы создания простой навигации, мы можем перейти к изучению [React Router API](https://github.com/reactjs/react-router/tree/master/docs).

>##### Помните, что существуют и другие библиотеки для навигации по приложению

>*Redux Router* — экспериментальная библиотека, она позволяет Вам хранить все состояние Вашего URL в хранилище redux. У нее то же самое API, что и у React Router, но сообщество и поддержка меньше, чем у React Router.

>*React Router Redux* создает связь между Вашим Redux-приложением и React Router и позволяет их синхронизировать. Без этой связи у Вас не будет возможности перемещаться по действиям при помощи Time Travel. Пока Вам это не требуется, React-router и Redux могут работать совершенно независимо.
