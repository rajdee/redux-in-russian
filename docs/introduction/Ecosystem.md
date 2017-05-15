# Экосистема

Redux — это небольшая библиотека, в которой соглашения и API были тщательно продуманы для удобного создания инструментов и подключения расширений экосистемы.

Мы рекомендуем ознакомиться с большим списком репозиториев, связанных с Redux в [Awesome Redux](https://github.com/xgrommx/awesome-redux). Он содержит примеры, макеты, промежуточное ПО (middleware), библиотеки утилит и т.д. [React/Redux Links](https://github.com/markerikson/react-redux-links) содержит руководства и другие полезные ресурсы для тех, кто изучает React или Redux и [Redux Ecosystem Links](https://github.com/markerikson/redux-ecosystem-links) содержит ссылки на многие библиотеки и дополнения, связанные с Redux.

На этой странице мы приводим только часть из данного списка репозиториев, функционал которых был лично проверен разработчиками Redux. Однако пусть это не ограничивает вас в экспериментах со сторонними модулями. Экосистема Redux растет очень быстро, но нам не хватает времени, чтобы отслеживать все изменения.

Итак, ознакомьтесь с нашим "избранным" и не стесняйтесь делиться своими наработками, если вы сделали что-то удивительное с Redux.

## Изучение Redux

### Скринкасты

* **[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)** — Learn the basics of Redux directly from its creator (30 free videos)
* **[Learn Redux](https://learnredux.com)** — Build a simple photo app that will simplify the core ideas behind Redux, React Router and React.js

### Примеры приложений

* [Official Examples](Examples.md) — A few official examples covering different Redux techniques
* [SoundRedux](https://github.com/andrewngu/sound-redux) — A SoundCloud client built with Redux
* [grafgiti](https://github.com/mohebifar/grafgiti) — Create graffiti on your GitHub contributions wall
* [React-lego](https://github.com/peter-mouland/react-lego) — How to plug into React, one block at a time.

### Туториалы и статьи

* [Redux Tutorial](https://github.com/happypoulp/redux-tutorial)
* [Redux Egghead Course Notes](https://github.com/tayiorbeii/egghead.io_redux_course_notes)
* [Integrating Data with React Native](http://makeitopen.com/tutorials/building-the-f8-app/data/)
* [What the Flux?! Let's Redux.](https://blog.andyet.com/2015/08/06/what-the-flux-lets-redux)
* [Leveling Up with React: Redux](https://css-tricks.com/learning-react-redux/)
* [A cartoon intro to Redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6)
* [Understanding Redux](http://www.youhavetolearncomputers.com/blog/2015/9/15/a-conceptual-overview-of-redux-or-how-i-fell-in-love-with-a-javascript-state-container)
* [Handcrafting an Isomorphic Redux Application (With Love)](https://medium.com/@bananaoomarang/handcrafting-an-isomorphic-redux-application-with-love-40ada4468af4)
* [Full-Stack Redux Tutorial](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html)
* [Getting Started with React, Redux, and Immutable](http://www.theodo.fr/blog/2016/03/getting-started-with-react-redux-and-immutable-a-test-driven-tutorial-part-2/)
* [Secure Your React and Redux App with JWT Authentication](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
* [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
* [Angular 2 — Introduction to Redux](https://medium.com/google-developer-experts/angular-2-introduction-to-redux-1cf18af27e6e)
* [Apollo Client: GraphQL with React and Redux](https://medium.com/apollo-stack/apollo-client-graphql-with-react-and-redux-49b35d0f2641)
* [Using redux-saga To Simplify Your Growing React Native Codebase](https://shift.infinite.red/using-redux-saga-to-simplify-your-growing-react-native-codebase-2b8036f650de)
* [Build an Image Gallery Using Redux Saga](http://joelhooks.com/blog/2016/03/20/build-an-image-gallery-using-redux-saga)
* [Working with VK API (in Russian)](https://www.gitbook.com/book/maxfarseer/redux-course-ru/details)

### Обсуждения

* [Live React: Hot Reloading and Time Travel](http://youtube.com/watch?v=xsSnOQynTHs) — See how constraints enforced by Redux make hot reloading with time travel easy
* [Cleaning the Tar: Using React within the Firefox Developer Tools](https://www.youtube.com/watch?v=qUlRpybs7_c) — Learn how to gradually migrate existing MVC applications to Redux
* [Redux: Simplifying Application State](https://www.youtube.com/watch?v=okdC5gcD-dM) — An intro to Redux architecture

## Использование Redux

### Адаптеры (Bindings)

* [react-redux](https://github.com/gaearon/react-redux) — React
* [ng-redux](https://github.com/wbuchwalter/ng-redux) — Angular
* [ng2-redux](https://github.com/wbuchwalter/ng2-redux) — Angular 2
* [backbone-redux](https://github.com/redbooth/backbone-redux) — Backbone
* [redux-falcor](https://github.com/ekosz/redux-falcor) — Falcor
* [deku-redux](https://github.com/troch/deku-redux) — Deku (6кб альтернатива для React)
* [polymer-redux](https://github.com/tur-nr/polymer-redux) - Polymer
* [ember-redux](https://github.com/toranb/ember-redux) - Ember.js

### Промежуточное ПО (Middleware)

* [redux-thunk](http://github.com/gaearon/redux-thunk) — Самый легкий способ создавать асинхронные генераторы действий (action creators) 
* [redux-promise](https://github.com/acdlite/redux-promise) — [FSA](https://github.com/acdlite/flux-standard-action)-совместимый promise middleware
* [redux-axios-middleware](https://github.com/svrcekmichal/redux-axios-middleware) — Redux middleware for fetching data with axios HTTP client
* [redux-observable](https://github.com/redux-observable/redux-observable/) — RxJS middleware for action side effects using "Epics"
* [redux-cycles](https://github.com/cyclejs-community/redux-cycles) — Handle Redux async actions using Cycle.js
* [redux-logger](https://github.com/fcomb/redux-logger) — Логирование каждого Redux действия (action) и следующего состояния (state)
* [redux-immutable-state-invariant](https://github.com/leoasis/redux-immutable-state-invariant) — Предупреждения об изменениях состояния во время разработки
* [redux-unhandled-action](https://github.com/socialtables/redux-unhandled-action) — Warns about actions that produced no state changes in development
* [redux-analytics](https://github.com/markdalgleish/redux-analytics) — Аналитика для Redux
* [redux-gen](https://github.com/weo-edu/redux-gen) — Generator middleware for Redux
* [redux-saga](https://github.com/yelouafi/redux-saga) — An alternative side effect model for Redux apps
* [redux-action-tree](https://github.com/cerebral/redux-action-tree) — Composable Cerebral-style signals for Redux
* [apollo-client](https://github.com/apollostack/apollo-client) — A simple caching client for any GraphQL server and UI framework built on top of Redux

### Маршрутизация (Routing)

* [react-router-redux](https://github.com/reactjs/react-router-redux) — Ruthlessly simple bindings to keep React Router and Redux in sync
* [redial](https://github.com/markdalgleish/redial) — Universal data fetching and route lifecycle management for React that works great with Redux

### Компоненты (Components)

* [redux-form](https://github.com/erikras/redux-form) — Поддержка Redux-состояний (stores) для html-форм, использующихся в React
* [react-redux-form](https://github.com/davidkpiano/react-redux-form) — Create forms easily in React with Redux

## Улучшения Reducer

* [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) — Настройка групповых и отложенных вызовов для подписавшихся на store
* [redux-history-transitions](https://github.com/johanneslumpe/redux-history-transitions) — переходы по History основанные на произвольных действиях (actions).
* [redux-optimist](https://github.com/ForbesLindesay/redux-optimist) — Оптимистичное использование действий (action) с возможностью их совершения или отмены в дальнейшем. 
> От переводчика: например, можно инициировать `ADD_TODO` action, state изменится, UI обновится мгновенно, а затем отправится запрос на сохранение на сервер. Если на сервере все прошло успешно, то все ок и больше ничего не произойдет. Но если же что-то пошло не так, то state будет "отмотан" до состояния когда `ADD_TODO` action еще не был применен, state изменится, UI снова обновится мгновенно. Т.е. UI будет выглядеть так, словно ничего и не произошло.
* [redux-optimistic-ui](https://github.com/mattkrick/redux-optimistic-ui) — A reducer enhancer to enable type-agnostic optimistic updates
* [redux-undo](https://github.com/omnidan/redux-undo) — Позволяет без усилий получить undo/redo-функциональность и историю действий для редюсеров
* [redux-ignore](https://github.com/omnidan/redux-ignore) — Ignore redux actions by array or filter function
* [redux-recycle](https://github.com/omnidan/redux-recycle) — Сброс состояния Redux для определенных действий
* [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) — Отправка нескольких действий с уведомлением одного абонента
* [redux-search](https://github.com/treasure-data/redux-search) — Automatically index resources in a web worker and search them without blocking
* [redux-electron-store](https://github.com/samiskin/redux-electron-store) — Store enhancers that synchronize Redux stores across Electron processes
* [redux-loop](https://github.com/raisemarketplace/redux-loop) — Sequence effects purely and naturally by returning them from your reducers
* [redux-side-effects](https://github.com/salsita/redux-side-effects) — Utilize Generators for declarative yielding of side effects from your pure reducers

### Утилиты (Utilities)

* [reselect](https://github.com/faassen/reselect) — Простая библиотека "селекторов", нашедшая вдохновение в геттерах NuclearJS
* [normalizr](https://github.com/gaearon/normalizr) — Упорядочивание вложенных ответов API для облегчения дальнейшего их использования в редюсерах
* [redux-actions](https://github.com/acdlite/redux-actions) — Уменьшение шаблонности в написании редюсеров и генераторов действий (action creators)
* [redux-act](https://github.com/pauldijou/redux-act) — An opinionated library for making reducers and action creators
* [redux-transducers](https://github.com/acdlite/redux-transducers) — Transducer-утилиты для Redux
* [redux-immutable](https://github.com/gajus/redux-immutable) — Used to create an equivalent function of Redux `combineReducers` that works with [Immutable.js](https://facebook.github.io/immutable-js/) state.
* [redux-tcomb](https://github.com/gcanti/redux-tcomb) — Иммутабельные, с проверкой типов состояния и действия для Redux
* [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) — Mock redux store for testing your app
* [redux-actions-assertions](https://github.com/dmitry-zaets/redux-actions-assertions) — Assertions for Redux actions testing
* [redux-bootstrap](https://github.com/remojansen/redux-bootstrap) — Bootstrapping function for Redux applications

### Инструменты разработчика (Developer Tools)

* [Redux DevTools](http://github.com/gaearon/redux-devtools) — Логирование действий с UI для путешествий во времени, горячая перезагрузка и обработка ошибок для редюсеров, [впервые представлено на React Europe](https://www.youtube.com/watch?v=xsSnOQynTHs)
* [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension) — Плагин для Chrome, по сути обертка для Redux DevTools с дополнительной функциональностью

### DevTools Monitors

* [Log Monitor](https://github.com/gaearon/redux-devtools-log-monitor) — The default monitor for Redux DevTools with a tree view
* [Dock Monitor](https://github.com/gaearon/redux-devtools-dock-monitor) — A resizable and movable dock for Redux DevTools monitors
* [Slider Monitor](https://github.com/calesce/redux-slider-monitor) — A custom monitor for Redux DevTools to replay recorded Redux actions
* [Inspector](https://github.com/alexkuz/redux-devtools-inspector) — A custom monitor for Redux DevTools that lets you filter actions, inspect diffs, and pin deep paths in the state to observe their changes
* [Diff Monitor](https://github.com/whetstone/redux-devtools-diff-monitor) — A monitor for Redux Devtools that diffs the Redux store mutations between actions
* [Filterable Log Monitor](https://github.com/bvaughn/redux-devtools-filterable-log-monitor/) — Filterable tree view monitor for Redux DevTools
* [Chart Monitor](https://github.com/romseguy/redux-devtools-chart-monitor) — A chart monitor for Redux DevTools
* [Filter Actions](https://github.com/zalmoxisus/redux-devtools-filter-actions) — Redux DevTools composable monitor with the ability to filter actions

### Общественные соглашения (Community Conventions)

* [Flux Standard Action](https://github.com/acdlite/flux-standard-action) — Дружелюбный стандарт для Flux action объектов
* [Canonical Reducer Composition](https://github.com/gajus/canonical-reducer-composition) — Слишком самоуверенный (opinionated) стандарт для структуры вложенных редюсеров
* [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) — Предложение по связыванию редюсеров, типов действий и действий

### Переводы

* [中文文档](http://camsong.github.io/redux-in-chinese/) — Chinese
* [繁體中文文件](https://github.com/chentsulin/redux) — Traditional Chinese
* [Redux in Russian](https://github.com/rajdee/redux-in-russian) — Russian
* [Redux en Español](http://es.redux.js.org/) - Spanish

## Прочее (More)

[Awesome Redux](https://github.com/xgrommx/awesome-redux) — обширный список репозиториев, имеющих отношение к Redux.
[React-Redux Links](https://github.com/markerikson/react-redux-links) is a curated list of high-quality articles, tutorials, and related content for React, Redux, ES6, and more.
[Redux Ecosystem Links](https://github.com/markerikson/redux-ecosystem-links) is a categorized collection of Redux-related libraries, addons, and utilities.

