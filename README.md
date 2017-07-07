# Redux-in-russian
Original [Redux](http://redux.js.org) documentation with a translation into Russian

Redux является предсказуемым контейнером состояния для JavaScript приложений.

Это позволяет вам создавать приложения, которые ведут себя одинаково в различных окружениях (клиент, сервер и нативные приложения), а также просто тестируются. Кроме того, это обеспечивает большой опыт отладки, например [редактирование кода в реальном времени в сочетании с time traveling](https://github.com/gaearon/redux-devtools).

Вы можете использовать Redux вместе с [React](https://facebook.github.io/react/) или с любой другой view-библиотекой. 
Это крошечная библиотека (2kB, включая зависимости).

[![build status](https://img.shields.io/travis/reactjs/redux/master.svg?style=flat-square)](https://travis-ci.org/reactjs/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![#rackt on freenode](https://img.shields.io/badge/irc-%23rackt%20%40%20freenode-61DAFB.svg?style=flat-square)](https://webchat.freenode.net/)
[![Changelog #187](https://img.shields.io/badge/changelog-%23187-lightgrey.svg?style=flat-square)](https://changelog.com/187)

>**Новинка! Изучайте Redux вместе с его создателем:  
>[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) (30 бесплатных видео)**

### Отзывы

>[“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)  
>Jing Chen, автор Flux

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)  
>Bill Fisher, автор документации Flux

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)  
>André Staltz, автор Cycle

### Опыт разработки

Я написал Redux пока работал над моим докладом на React Europe, который назывался [“Hot Reloading with Time Travel”](https://www.youtube.com/watch?v=xsSnOQynTHs). Моей целью было создание библиотеки управления состоянием с минимальным API, но вполне предсказуемым поведением, так чтобы можно было реализовать протоколирование, горячую перезагрузку, путешествия во времени, универсальные приложения, запись и воспроизведение, без каких-либо вложений от разработчика.

### Влияния

Redux развивает идеи [Flux](http://facebook.github.io/flux/), но избегает его сложности, воспользовавшись примерами из [Elm](https://github.com/evancz/elm-architecture-tutorial/).
Вне зависимости от того, использовали вы их или нет, Redux занимает всего несколько минут, чтобы начать с ним работу.

### Установка

Для установки стабильной версии:

```
npm install --save redux
```

Скорее всего, вам также понадобится [связка с React](https://github.com/reactjs/react-redux) и [инструменты разработчика](https://github.com/gaearon/redux-devtools).

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

Предполагается, что вы используете пакетный менеджер [npm](https://www.npmjs.com/) вместе со сборщиком модулей, типа [Webpack](http://webpack.github.io) или [Browserify](http://browserify.org/), использующие [CommonJS модули](http://webpack.github.io/docs/commonjs.html).

Если вы еще не используете [npm](https://www.npmjs.com/) или современнные сборщики модулей и предпочитаете сборку [UMD](https://github.com/umdjs/umd), что делает `Redux` доступным в качестве глобального объекта, вы можете использовать предкомпилированную версию с [cdnjs](https://cdnjs.com/libraries/redux). Мы *не* рекомендуем этот подход для любого серьезного применения, т.к. большинство библиотек используемых с  Redux доступны только на [npm](https://www.npmjs.com/).

### Основные принципы

Все состояние вашего приложения сохранено в объекте внутри одного *хранилища (store)*. 
Единственный способ изменить дерево состояния - это вызвать *действие (action)*, объект описывающий то, что случилось. 
Чтобы указать, каким образом действия преобразовывают дерево состояния, вы пишете чистые "редюсеры".

Все, готово!

```js
import { createStore } from 'redux'

/**
 * Это редюсер - чистая функция в формате(state, action) => state.
 * Он описывает то, как действие преобразовывает состояние в следующее состояние 
 *
 * Формат состояния зависит от вас: это может быть примитивом,
 * массивом, объектом
 * или даже структурой данных Immutable.js.
 * Важно только одно, вы не должны изменять объект состояния
 * напрямую, а возвращать
 * новый объект, если состояние изменилось
 *
 * В этом примере мы используем `switch` и строки, но вы можете
 * использовать хелпер, который следует другому соглашению
 * (например, function maps), если это имеет смысл для вашего
 * проекта.
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Создаем хранилище Redux которое хранит состояние вашего приложения.
// Его API - { subscribe, dispatch, getState }.
let store = createStore(counter)

// Вы можете подписаться на обновления вручную или использовать биндинги к вашему view layer.
store.subscribe(() =>
  console.log(store.getState())
)

// Единственный способ изменить внутреннее состояние - это вызвать действие
// Действия могут быть сериализированы, залогированы или сохранены и далее воспроизведены.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Вместо того, что бы изменять состояние напрямую, вы определяете изменения, которые вы хотите произвести, с помощью простых объектов называемых *действия*. Затем вы пишете специальную функцию, называемую *редюсер*, чтобы решить, каким образом каждое действие преобразует все состояние приложения.

Если вы пришли из Flux, есть одно важное различие, которое вы должны понимать. Redux не имеет Диспетчера (Dispatcher) или поддержки множества хранилищ. Вместо этого есть только одно хранилище с одной корневой функцией-редюсером. Когда ваше приложение разрастется, вместо добавления хранилищ, вы разделяете корневой редюсер на более мелкие редюсеры, которые независимо друг от друга обслуживают разные части дерева состояния. Это аналогично тому, что в React приложении есть только один корневой компонент, состоящий из множества мелких компонентов.

Эта архитектура может показаться излишней для счетчика приложения, но красота этого шаблона в том, насколько хорошо она масштабируется для больших и сложных приложений. Она также предоставляет очень мощные инструменты для разработчиков, позволяющих проследить каждую мутацию к действию, вызвавшему его. Вы можете записывать пользовательские сессии и воспроизводить их просто переигрывая каждое действие.

### Изучайте Redux вместе с его создателем

[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) - это видео-курс, состоящий из 30 роликов созданных Дэном Абрамовым, автором Redux. Он предназначен для дополнения части документации «Основы», привнося дополнительные сведения о неизменности, тестировании, лучших практиках Redux, а также об использовании Redux с React. **Данный курс является и всегда будет бесплатным.**

>[“Great course on egghead.io by @dan_abramov - instead of just showing you how to use #redux, it also shows how and why redux was built!”](https://twitter.com/sandrinodm/status/670548531422326785)  
>Sandrino Di Mattia

>[“Plowing through @dan_abramov 'Getting Started with Redux' - its amazing how much simpler concepts get with video.”](https://twitter.com/chrisdhanaraj/status/670328025553219584)  
>Chris Dhanaraj

>[“This video series on Redux by @dan_abramov on @eggheadio is spectacular!”](https://twitter.com/eddiezane/status/670333133242408960)  
>Eddie Zaneski

>[“Come for the name hype. Stay for the rock solid fundamentals. (Thanks, and great job @dan_abramov and @eggheadio!)”](https://twitter.com/danott/status/669909126554607617)  
>Dan

>[“This series of videos on Redux by @dan_abramov is repeatedly blowing my mind - gunna do some serious refactoring”](https://twitter.com/gelatindesign/status/669658358643892224)  
>Laurence Roberts

И так, чего же вы ждете?

#### [Посмотрите 30 бесплатных уроков!](https://egghead.io/series/getting-started-with-redux)

Если вам понравился мой курс, подумайте о поддержке Egghead путем [покупки подписки] (https://egghead.io/pricing). Подписчики имеют доступ к исходному коду, для примера, в каждом из моих видео, а также к массе продвинутых уроков по другим темам, включая JavaScript in depth, React, Angular и многое другое. Многие [преподаватели Egghead] (https://egghead.io/instructors) также являются авторами библиотек с открытым исходным кодом, т.ч. покупка подписки - это хороший способ поблагодарить их за работу, которую они сделали.


### Документация

* [Введение](https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/index.html)
* [Основы](https://rajdee.gitbooks.io/redux-in-russian/content/docs/basics/index.html)
* [Продвинутое использование](https://rajdee.gitbooks.io/redux-in-russian/content/docs/advanced/index.html)
* [Рецепты](https://rajdee.gitbooks.io/redux-in-russian/content/docs/recipes/index.html)
* [Поиск неисправностей](https://rajdee.gitbooks.io/redux-in-russian/content/docs/Troubleshooting.html)
* [Глоссарий](https://rajdee.gitbooks.io/redux-in-russian/content/docs/Glossary.html)
* [Справочник по API](https://rajdee.gitbooks.io/redux-in-russian/content/docs/api/index.html)

Для экпорта в PDF, ePub MOBI или чтения в оффлайн и инструкциям, как это можно осуществить, обратите внимание на: [paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs).

### Примеры

* [Counter Vanilla](http://redux.js.org/docs/introduction/Examples.html#counter-vanilla) ([source](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla))
* [Counter](http://redux.js.org/docs/introduction/Examples.html#counter) ([source](https://github.com/reactjs/redux/tree/master/examples/counter))
* [Todos](http://redux.js.org/docs/introduction/Examples.html#todos) ([source](https://github.com/reactjs/redux/tree/master/examples/todos))
* [Todos with Undo](http://redux.js.org/docs/introduction/Examples.html#todos-with-undo) ([source](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo))
* [TodoMVC](http://redux.js.org/docs/introduction/Examples.html#todomvc) ([source](https://github.com/reactjs/redux/tree/master/examples/todomvc))
* [Shopping Cart](http://redux.js.org/docs/introduction/Examples.html#shopping-cart) ([source](https://github.com/reactjs/redux/tree/master/examples/shopping-cart))
* [Tree View](http://redux.js.org/docs/introduction/Examples.html#tree-view) ([source](https://github.com/reactjs/redux/tree/master/examples/tree-view))
* [Async](http://redux.js.org/docs/introduction/Examples.html#async) ([source](https://github.com/reactjs/redux/tree/master/examples/async))
* [Universal](http://redux.js.org/docs/introduction/Examples.html#universal) ([source](https://github.com/reactjs/redux/tree/master/examples/universal))
* [Real World](http://redux.js.org/docs/introduction/Examples.html#real-world) ([source](https://github.com/reactjs/redux/tree/master/examples/real-world))

Если вы новичок в экосистеме NPM и имеете проблемы с получением и запуском проекта или не уверены, куда вставить шаблон, попробуйте [simplest-redux-example](https://github.com/jackielii/simplest-redux-example) который использует Redux вместе с React и Browserify.

### Обсуждения

Подключайтесь к каналу [#redux](https://discord.gg/0ZcbPKXt5bZ6au5t) находящегося в Discord сообществе [Reactiflux](http://www.reactiflux.com).

### Благодарности

* [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) за великолепное введение в моделирование обновления состояния посредством редюсеров;
* [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) за взрыв моего сознания;
* [Developing ClojureScript with Figwheel](https://www.youtube.com/watch?v=j-kj2qwJa_E) за убеждение меня в том, что переоценка должна "просто работать";
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) за Hot Module Replacement;
* [Flummox](https://github.com/acdlite/flummox) за обучение меня подходу Flux без шаблонов или синглетонов;
* [disto](https://github.com/threepointone/disto) за доказательство концепции "hot reloadable" хранилищ;
* [NuclearJS](https://github.com/optimizely/nuclear-js) за доказательство того, что такая архитектура может быть производительной;
* [Om](https://github.com/omcljs/om) за популяризацию идеи одного атома состояния;
* [Cycle](https://github.com/cyclejs/cycle-core) за демонстрацию того, как часто функция является лучшим инструментом;
* [React](https://github.com/facebook/react) за прагматические инновации.

Особенная благодарность [Jamie Paton](http://jdpaton.github.io) за передачу прав на `redux` NPM пакет.

### История изменений

Это проект придерживается принципов [семантического версионирования](http://semver.org/).  
Каждый релиз, вместе с инструкциями миграции, документированы на странице [релизов](https://github.com/reactjs/redux/releases) Github.

### Меценаты

Разработка Redux была [профинансирована сообществом](https://www.patreon.com/reactdx).  
Познакомьтесь с некоторыми из выдающихся компаний, которые сделали это возможным:

* [Webflow](https://github.com/webflow)
* [Ximedes](https://www.ximedes.com/)

[См. полный список меценатов Redux.](PATRONS.md)

### License

MIT
