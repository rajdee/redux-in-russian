# Redux-in-russian
Оригинальная документация по [Redux](http://redux.js.org) с переводом на русский

# <a href='http://redux.js.org'><img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60' alt='Redux Logo' aria-label='Redux.js.org' /></a>

Redux - это предсказуемое хранилище состояния для JavaScript приложений.
(Не путайте с WordPress фреймворком – [Redux Framework](https://reduxframework.com/).)

Он помогает вам писать приложения, которые ведут себя предсказуемо, работают в разных окружениях (клиентских, серверных и нативные приложения) и которые легко тестировать. Кроме того, он предоставляет отличные возможности для разработчиков, такие как [редактирование кода в реальном времени в сочетании с отладчиком путешествий во времени](https://github.com/reduxjs/redux-devtools).

Вы можете использовать Redux вместе с [React](https://facebook.github.io/react/) или с любой другой view-библиотекой. 
Это крошечная библиотека (2kB, включая зависимости).

[![build status](https://img.shields.io/travis/reduxjs/redux/master.svg?style=flat-square)](https://travis-ci.org/reduxjs/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![Changelog #187](https://img.shields.io/badge/changelog-%23187-lightgrey.svg?style=flat-square)](https://changelog.com/187)

## Установка

```
npm install @reduxjs/toolkit react-redux
```

Для более подробной информации, смотри [страницу установки](https://redux.js.org/introduction/installation).

## Документация

Документация по Redux находится по адресу **https://redux.js.org**:

- [Введение](https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/GettingStarted.html)
- [Рецепты](https://rajdee.gitbooks.io/redux-in-russian/content/docs/recipes/index.html)
- [FAQ](https://rajdee.gitbooks.io/redux-in-russian/content/docs/FAQ.html)
- [Справочник по API](https://rajdee.gitbooks.io/redux-in-russian/content/docs/api/index.html)

Для экпорта в PDF, ePub MOBI для чтения в оффлайн и инструкциям, как это можно осуществить, обратите внимание на:  [paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs).

Для документации в Offline, пожалуйста посмотрите: [devdocs](http://devdocs.io/redux/)

## Изучите Redux

### Учебное пособие по основам Redux

[**Redux Essentials tutorial**](https://redux.js.org/tutorials/essentials/part-1-overview-concepts) - это пособие по основам, которое учит вас, как использовать Redux правильно, используя наши последние рекомендуемые API и лучшие практики. Мы рекомендуем начать с этого.

### Дополнительные учебные пособия

- Документация Redux [**Базовое пособие**](https://rajdee.gitbooks.io/redux-in-russian/content/docs/basics/) и [**Продвинутое пособие**](https://rajdee.gitbooks.io/redux-in-russian/content/docs/advanced/) - это руководства, которые объясняют, как работает Redux, начиная с простых принципов.
- Репозиторий Redux, содержит несколько примеров проектов, демонстрирующих  различные аспекты использования Redux. Почти во всех примерах есть соответствующая песочница CodeSandbox. Это интерактивная версия кода, которую можно попробовать онлайн. Смотрите полный список на **[странице Примеров](https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/Examples.html)**.
- видеокурсы на Egghead.io, от создателя Redux Dan Abramov **бесплатный  ["Getting Started with Redux"](https://egghead.io/series/getting-started-with-redux)** и **[Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)** 
- от разработчика Redux Mark Erikson - доклад с конференции **["Redux Fundamentals"](http://blog.isquaredsoftware.com/2018/03/presentation-reactathon-redux-fundamentals/)** и слайды воркшопа [**"Redux Fundamentals"**](https://blog.isquaredsoftware.com/2018/06/redux-fundamentals-workshop-slides/)
- Пост Dave Ceddia [**A Complete React Redux Tutorial for Beginners**](https://daveceddia.com/redux-tutorial/)

### Помощь и обсуждения

**[Канал #redux](https://discord.gg/0ZcbPKXt5bZ6au5t)**  как часть **[Reactiflux Discord community](http://www.reactiflux.com)**, наш официальный ресурс для всех вопросов связанных с изучением и использованием Redux. Reactiflux - отличное место для того, чтобы общаться, задавать вопросы и учиться - присоединяйтесь к нам!


## Прежде чем продолжить

Redux - это ценный инструмент для управления вашим состоянием, но вы также должны учитывать, подходит ли он для вашей ситуации. Не используйте Redux только потому, что кто-то сказал, что вам нужно - потратьте некоторое время, чтобы понять потенциальные выгоды и компромиссы с его использованием.

Вот несколько советов о том, когда имеет смысл использовать Redux:
- У вас есть обоснованные объемы данных, меняющихся со временем
- Вам нужен один источник информации для вашего состояния
- Вы приходите к выводу, что сохранять все ваше состояние в компоненте верхнего уровня уже недостаточно

Да, эти рекомендации являются субъективными и расплывчатыми, но это справедливо. Момент, в который вы должны интегрировать Redux в ваше приложение, отличается для каждого пользователя и каждого приложения.

>**Дополнительные сведения о том, как использовать Redux, см:**
>
> - **[You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)**
> - **[The Tao of Redux, Part 1 - Implementation and Intent](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/)**
> - **[The Tao of Redux, Part 2 - Practice and Philosophy](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/)**
> - **[Redux FAQ](https://redux.js.org/faq)**

## Опыт разработки

Дэн Абрамов, автор Redux, написал Redux пока работал над своим докладом на React Europe, который назывался [“Hot Reloading with Time Travel”](https://www.youtube.com/watch?v=xsSnOQynTHs). Его целью было создание библиотеки управления состоянием с минимальным API, но вполне предсказуемым поведением, так чтобы можно было реализовать протоколирование, горячую перезагрузку, путешествия во времени, универсальные приложения, запись и воспроизведение, без каких-либо вложений от разработчика.

## Влияния

Redux развивает идеи [Flux](http://facebook.github.io/flux/), но избегает его сложности, воспользовавшись примерами из [Elm](https://github.com/evancz/elm-architecture-tutorial/).
Даже если вы не использовали Flux или Elm, для начала работы с Redux потребуется всего несколько минут.

## Основные принципы

Все состояние вашего приложения сохранено в объекте внутри одного *стора (store)*. 
Единственный способ изменить дерево состояния - это вызвать экшен (action)*, объект описывающий то, что случилось. 
Чтобы указать, каким образом экшены преобразовывают дерево состояния, вы пишете чистые "редьюсеры".

Все, готово!

```js
import { createStore } from 'redux'

/**
 * Это редьюсер - чистая функция в формате (state, action) => state.
 * Он описывает то, как экшен преобразовывает состояние в следующее состояние 
 *
 * Структура состояния зависит от вас: это может быть примитивом,
 * массивом, объектом или даже структурой данных Immutable.js.
 * Важно только одно, вы не должны изменять объект состояния
 * напрямую, а возвращать новый объект, если состояние изменилось
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

// Создаем Redux стор, который хранит состояние вашего приложения.
// Его API - { subscribe, dispatch, getState }.
let store = createStore(counter)

// Вы можете использовать subscribe() для обновления UI в ответ на изменения состояния.
// Обычно вы должны использовать библиотеку привязки (например, React Redux), а не использовать subscribe() напрямую.
// Однако также может быть полезно сохранить текущее состояние в localStorage.
store.subscribe(() => console.log(store.getState()))


// Единственный способ изменить внутреннее состояние - это вызвать экшен
// Экшены могут быть сериализированы, залогированы или сохранены и далее воспроизведены.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Вместо того, что бы изменять состояние напрямую, вы определяете изменения, которые вы хотите произвести, с помощью простых объектов называемых _экшенами_. Затем вы пишете специальную функцию, называемую _редьюсер_, чтобы решить, каким образом каждый экшен преобразует все состояние приложения.

Эта архитектура может показаться излишней для счетчика приложения, но красота этого шаблона в том, насколько хорошо она масштабируется для больших и сложных приложений. Она также предоставляет очень мощные инструменты для разработчиков, позволяющих проследить каждую мутацию к экшену, вызвавшему его. Вы можете записывать пользовательские сессии и воспроизводить их просто переигрывая каждый экшена.

## Примеры

Почти все примеры имеют соответствующую песочницу CodeSandbox. Это интерактивная версия кода, с которой вы можете играть онлайн.

- [**Counter Vanilla**](https://redux.js.org/introduction/examples#counter-vanilla): [Source](https://github.com/reduxjs/redux/tree/master/examples/counter-vanilla)
- [**Counter**](https://redux.js.org/introduction/examples#counter): [Source](https://github.com/reduxjs/redux/tree/master/examples/counter) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/counter)
- [**Todos**](https://redux.js.org/introduction/examples#todos): [Source](https://github.com/reduxjs/redux/tree/master/examples/todos) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todos)
- [**Todos with Undo**](https://redux.js.org/introduction/examples#todos-with-undo): [Source](https://github.com/reduxjs/redux/tree/master/examples/todos-with-undo) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todos-with-undo)
- [**Todos w/ Flow**](https://redux.js.org/introduction/examples#todos-flow): [Source](https://github.com/reduxjs/redux/tree/master/examples/todos-flow)
- [**TodoMVC**](https://redux.js.org/introduction/examples#todomvc): [Source](https://github.com/reduxjs/redux/tree/master/examples/todomvc) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todomvc)
- [**Shopping Cart**](https://redux.js.org/introduction/examples#shopping-cart): [Source](https://github.com/reduxjs/redux/tree/master/examples/shopping-cart) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/shopping-cart)
- [**Tree View**](https://redux.js.org/introduction/examples#tree-view): [Source](https://github.com/reduxjs/redux/tree/master/examples/tree-view) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/tree-view)
- [**Async**](https://redux.js.org/introduction/examples#async): [Source](https://github.com/reduxjs/redux/tree/master/examples/async) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/async)
- [**Universal**](https://redux.js.org/introduction/examples#universal): [Source](https://github.com/reduxjs/redux/tree/master/examples/universal)
- [**Real World**](https://redux.js.org/introduction/examples#real-world): [Source](https://github.com/reduxjs/redux/tree/master/examples/real-world) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/real-world)

Если вы новичок в экосистеме NPM и имеете проблемы с получением и запуском проекта или не уверены, куда вставить шаблон, попробуйте [simplest-redux-example](https://github.com/jackielii/simplest-redux-example) который использует Redux вместе с React и Browserify.

## Отзывы

>[“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen, автор Flux

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher, автор документации Flux

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz, creator of Cycle


## Благодарности

* [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) за великолепное введение в моделирование обновления состояния посредством редьюсеров;
* [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) за взрыв моего сознания;
* [Developing ClojureScript with Figwheel](https://www.youtube.com/watch?v=j-kj2qwJa_E) за убеждение меня в том, что переоценка должна "просто работать";
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) за Hot Module Replacement;
* [Flummox](https://github.com/acdlite/flummox) за обучение меня подходу Flux без шаблонов или синглетонов;
* [disto](https://github.com/threepointone/disto) за доказательство концепции "hot reloadable" сторов;
* [NuclearJS](https://github.com/optimizely/nuclear-js) за доказательство того, что такая архитектура может быть производительной;
* [Om](https://github.com/omcljs/om) за популяризацию идеи одного атома состояния;
* [Cycle](https://github.com/cyclejs/cycle-core) за демонстрацию того, как часто функция является лучшим инструментом;
* [React](https://github.com/facebook/react) за прагматические инновации.

Особенная благодарность [Jamie Paton](http://jdpaton.github.io) за передачу прав на `redux` NPM пакет.

## Логотип

Вы можете найти официальное лого [на GitHub](https://github.com/reactjs/redux/tree/master/logo).

## История изменений

Это проект придерживается принципов [семантического версионирования](http://semver.org/).
Каждый релиз, вместе с инструкциями миграции, документированы на странице [релизов](https://github.com/reactjs/redux/releases) Github.

## Меценаты

Разработка Redux была [профинансирована сообществом](https://www.patreon.com/reactdx).
Познакомьтесь с некоторыми из выдающихся компаний, которые сделали это возможным:

* [Webflow](https://github.com/webflow)
* [Ximedes](https://www.ximedes.com/)

[См. полный список меценатов Redux.](PATRONS.md), а также постоянно растущий список [людей и компаний, которые используют Redux](https://github.com/reactjs/redux/issues/310).

## License

[MIT](LICENSE.md)
