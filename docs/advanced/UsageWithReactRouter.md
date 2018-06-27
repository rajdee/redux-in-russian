# Использование с React Router

Итак, вы хотите использовать маршрутизацию с
Вашим Redux-приложением. Для этого вы можете использовать
[React Router](https://github.com/reactjs/react-router).
Тогда Redux будет источником правдивых данных, а React
Router — единственным источником URL. В большинстве случаев,
***правильно*** разделять эти понятия, но до тех пор, пока
вам не понадобится путешествовать во времени (Time Travel)
и перематывать действия, которые изменяют URL.

> Стоит также отметить различия в версиях react-route 2/3 и 4.
> Для более полного понимания различий не лишним 
> будет ознакомиться с руководством по миграции:
> [Migrating from v2/v3 to v4](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/guides/migrating.md)

## React Router, React Router DOM и React Router Native

На текущий момент экосистема `react-router@^4.3` представлена пятью пакетами:
1) `react-router` инкапсулирует общие части для `react-router-dom` и `react-router-native`;
2) `react-router-dom` предатавляет собой DOM-обвязки для `react-router`;
3) `react-router-native` - привязка к React Native;
4) `react-router-redux` - deprecated, вместо него рекомендуется к использованию [Connected React Router](https://github.com/supasate/connected-react-router);
5) `react-router-config` - статическая конфигурация React Routes (и не только).

Для текущего примера достаточно установить `react-router-dom`, пакет также экспортирует `react-router`.

## Установка React Router DOM
`react-router-dom` доступно в npm . В этом руководстве преполагается использование версии `react-router@^4.3`.

`npm install --save react-router-dom`

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

Во-первых, нам надо импортировать `<BrowserRouter />` и `<Route />` из React Router. Вот как это делается:

```js
import { BrowserRouter, Route } from 'react-router-dom';
```

В React-приложении обычно `<Route />` оборачивается в `<BrowserRouter />`, так что когда URL меняется, `<BrowserRouter />` находит часть своих роутов и рендерит их сформированные компоненты. `<Route />` используется для (неофициального) сопоставления роутов иерархии компонентов вашего приложения. Вы можете объявить в `path` путь, используемый в URL, и в `component` — компонент, который должен быть сгенерирован при совпадении с этим URL.

```js
const Root = () => (
  <BrowserRouter>
    <div>
      <Route path="/" component={App} />
    </div>
  </BrowserRouter>
);
```

> Учтите, что **BrowserRouter должен иметь только один дочерний элемент**

Однако, в нашем Redux-приложении нам все еще требуется `<Provider />` — компонент высшего порядка, поставляемый с React Redux, который позволяет привязать Redux к React (см. [Использование с React](../basics/UsageWithReact.md)).

Далее мы импортируем `<Provider />` из React Redux:

```js
import { Provider } from 'react-redux';
```

Обернем `<BrowserRouter />` в `<Provider />`, таким образом обработчики маршрутизации смогут получить [доступ к `хранилищу`](../basics/UsageWithReact.html#передаем-хранилище).

```js
const Root = ({ store }) => (
  <Provider store={store}>
    <BrowserRouter>
      <div>
        <Route path="/" component={App} />
      </div>
    </BrowserRouter>
  </Provider>
);
```

Теперь компонент `<App />` будет отрендерен, если URL содержит '/'. Кроме того, мы будем добавлять необязательный `:filter` параметр к `/`. Нам это потребуется позже, когда мы будем читать параметр `:filter` из URL.

```js
<Route path="/:filter" component={App} />
```

Полный код компонента

#### `components/Root.js`
``` js
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { BrowserRouter, Route } from 'react-router-dom';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <BrowserRouter>
      <div>
        <Route path="/:filter" component={App} />
      </div>
    </BrowserRouter>
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
import { Link } from 'react-router-dom';

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
const App = (props) => {
  return (
    <div>
      <AddTodo />
      <VisibleTodoList
        filter={props.match.params.filter || 'all'}
      />
      <Footer />
    </div>
  );
};
```

## Следующие шаги

Теперь, когда мы знаем основы создания простой навигации, мы можем перейти к изучению [React Router API](https://reacttraining.com/react-router/).

>##### Помните, что существуют и другие библиотеки для навигации по приложению

>*Redux Router* — экспериментальная библиотека, она позволяет Вам хранить все состояние Вашего URL в хранилище redux. У нее то же самое API, что и у React Router, но сообщество и поддержка меньше, чем у React Router.

>*React Router Redux* создает связь между Вашим Redux-приложением и React Router и позволяет их синхронизировать. Без этой связи у Вас не будет возможности перемещаться по действиям при помощи Time Travel. Пока Вам это не требуется, React-router и Redux могут работать совершенно независимо.
