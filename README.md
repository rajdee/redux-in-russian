# Redux-in-russian
Original [Redux](http://redux.js.org) documentation with a translation into Russian

# <a href='http://redux.js.org'><img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60' alt='Redux Logo' aria-label='Redux.js.org' /></a>

Redux является предсказуемым контейнером состояния для JavaScript приложений.
(Не путайте с WordPress фреймворком – [Redux Framework](https://reduxframework.com/).)

Это позволяет вам создавать приложения, которые ведут себя одинаково в различных окружениях (клиент, сервер и нативные приложения), а также просто тестируются. Кроме того, это обеспечивает большой опыт отладки, например [редактирование кода в реальном времени в сочетание с time traveling](https://github.com/gaearon/redux-devtools).

Вы можете использовать Redux вместе с [React](https://facebook.github.io/react/) или с любой другой view-библиотекой. 
Это крошечная библиотека (2kB, включая зависимости).

[![build status](https://img.shields.io/travis/reactjs/redux/master.svg?style=flat-square)](https://travis-ci.org/reactjs/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![Changelog #187](https://img.shields.io/badge/changelog-%23187-lightgrey.svg?style=flat-square)](https://changelog.com/187)


## Изучите Redux

У нас есть множество доступных ресурсов, которые помогут вам изучить Redux, независимо от того, каков ваш уровень знаний или стиль обучения.

### Только основы

Если вы новичок в Redux и хотите понять основные понятия, см:

- **[Базовое руководство в документации Redux](https://github.com/rajdee/redux-in-russian/tree/master/docs/basics)**
- **Бесплатная видео-серия ["Getting Started with Redux"](https://egghead.io/series/getting-started-with-redux)** на Egghead.io от создателя Redux Дэна Абрамова
- Слайдшоу **["Redux Fundamentals"](http://blog.isquaredsoftware.com/2018/03/presentation-reactathon-redux-fundamentals/)** а также **[список полезных ресурсов для изучения Redux](http://blog.isquaredsoftware.com/2017/12/blogged-answers-learn-redux/)** от одного из разработчиков Redux - Марка Эриксона
- Если вы лучше учитесь просматривая код и играя с ним, ознакомьтесь с нашим списком **[примеров Redux приложений](https://redux.js.org/introduction/examples)**, доступных в качестве отдельных проектов в репозитории Redux, а также в качестве интерактивных примеров на CodeSandbox.
- **[Руководства по Redux](https://github.com/markerikson/react-redux-links/blob/master/redux-tutorials.md)** в секции **[списка ссылок React/Redux](https://github.com/markerikson/react-redux-links)**.  Вот список рекомендуемых нами учебников:
    - Посты Dave Ceddia [What Does Redux Do? (and when should you use it?)](https://daveceddia.com/what-does-redux-do/) и [How Redux Works: A Counter-Example](https://daveceddia.com/how-does-redux-work/) являются отличным введением к основам Redux и как использовать его с React, как и этот пост [React and Redux: An Introduction](http://jakesidsmith.com/blog/post/2017-11-18-redux-and-react-an-introduction/).
    - Пост Valentino Gagliardi [React Redux Tutorial for Beginners: Learning Redux in 2018](https://www.valentinog.com/blog/react-redux-tutorial-beginners/) является отличным расширенным введением во многие аспекты использования Redux.
    - Статья на CSS Tricks [Leveling Up with React: Redux](https://css-tricks.com/learning-react-redux/) хорошо раскрывает основы Redux.
    - Это руководство [DevGuides: Introduction to Redux](http://devguides.io/redux/) раскрывает различные аспекты Redux, включая экшены, редюсеры, мидлвары, использование с React.


### Средний уровень

Когда вы освоите основы работы с действиями, редюсерами и хранилищем, у вас могут возникнуть вопросы по таким темам, как работа с асинхронной логикой и AJAX запросами, подключение UI фреймворка, например React, к вашему хранилищу Redux и настройка приложения для использования Redux:

- **[Раздел документации "Продвинутое использование"](https://github.com/rajdee/redux-in-russian/tree/master/docs/advanced)** раскрывает работу с асинхронной логикой, мидлвар, маршрутизацию.
- Страница документации **["Образовательные ресурсы"](https://github.com/rajdee/redux-in-russian/tree/master/docs/introduction/learning-resources)** указывает на рекомендуемые статьи по различным темам, связанным с Redux.
- Серия статей из 8 частей от Sophie DeBenedetto **[Building a Simple CRUD App with React + Redux](http://www.thegreatcodeadventure.com/building-a-simple-crud-app-with-react-redux-part-1/)** показывает, как собрать базовое приложение CRUD с нуля.


### Использование в реальном мире

Переход от приложения TodoMVC, к реальному production приложению, может стать большим скачком, но у нас есть много ресурсов, которые помогут:

- **[Бесплатная видеосерия "Building React Applications with Idiomatic Redux"](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)** от создателя Redux Дэна Абрамова, основывается на его первой серии видео и охватывает такие темы, как мидлвары, маршрутизация и постоянство.
- **[Redux FAQ](docs/FAQ.md)** отвечает на многие базовые вопросы, связанные с использованием Redux, а также **[секция документации "Рецепты"](https://github.com/rajdee/redux-in-russian/tree/master/docs/recipes)** которая содержит информацию об обработке полученных данных, тестирование, структурирование логики редюсеров и уменьшении шаблонности.
- **[Серия руководств "Practical Redux"](http://blog.isquaredsoftware.com/series/practical-redux/)** от одного из разработчиков Redux Марка Эриксона, демонстрирует практические техники среднего и продвинутого уровня для работы с React и Redux (также доступен в виде **[интерактивного курса на Educative.io](https://www.educative.io/collection/5687753853370368/5707702298738688)**)
-**[Список ссылок React/Redux](https://github.com/markerikson/react-redux-links)** содержит классифицированные статьи о работе с [редюсерами и селекторами](https://github.com/markerikson/react-redux-links/blob/master/redux-reducers-selectors.md), [управление сайд-эффектами](https://github.com/markerikson/react-redux-links/blob/master/redux-side-effects.md), [архитектуре и лучшим практикам Redux](https://github.com/markerikson/react-redux-links/blob/master/redux-architecture.md), и т.д.
- Наше сообщество создало тысячи библиотек, дополнений и инструментов, связанных с Redux. **[Страница документации "Экосистема"](https://github.com/rajdee/redux-in-russian/blob/master/docs/introduction/Ecosystem.md)** перечисляет наши рекомендации и есть полный список, доступный в **[Redux addons catalog](https://github.com/markerikson/redux-ecosystem-links)**.
- Если вы хотите учиться на реальной кодовой базе приложений, каталог дополнений также содержит список **[специально написанных и реальных приложений](https://github.com/markerikson/redux-ecosystem-links/blob/master/apps-and-examples.md)**.

Наконец, Марк Эриксон проводит серию **[воркшопов по Redux при помощи Workshop.me](#redux-workshops)**. Проверьте [расписание воркшопов](https://workshop.me/?a=mark) для предстоящих дат и местоположений.


### Помощь и обсуждения

**[Канал #redux](https://discord.gg/0ZcbPKXt5bZ6au5t)**, как часть **[Reactiflux Discord community](http://www.reactiflux.com)**, наш официальный ресурс для всех вопросов связанных с изучением и использованием Redux. Reactiflux - отличное место для того, чтобы общаться, задавать вопросы и учиться - присоединяйтесь к нам!




## Прежде чем продолжить

Redux - это ценный инструмент для управления вашим состоянием, но вы также должны учитывать, подходит ли он для вашей ситуации. Не используйте Redux только потому, что кто-то сказал, что вам нужно - потратьте некоторое время, чтобы понять потенциальные выгоды и компромиссы с его использованием.

Вот несколько советов о том, когда имеет смысл использовать Redux:
* У вас есть обоснованные объемы данных, меняющихся со временем
* Вам нужен один источник информации для вашего состояния
* Вы приходите к выводу, что сохранять все ваше состояние в компоненте верхнего уровня уже недостаточно

Да, эти рекомендации являются субъективными и расплывчатыми, но это справедливо. Точка, в которой вы должны интегрировать Redux в ваше приложение, отличается для каждого пользователя и каждого приложения.

>**Дополнительные сведения о том, как использовать Redux, см:**<br>
>- **[You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)**<br>
>- **[The Tao of Redux, Part 1 - Implementation and Intent](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/)**<br>
>- **[The Tao of Redux, Part 2 - Practice and Philosophy](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/)**
>- **[Redux FAQ](https://redux.js.org/faq)**

### Опыт разработки

Дэн Абрамов, автор Redux, написал Redux пока работал над своим докладом на React Europe, который назывался [“Hot Reloading with Time Travel”](https://www.youtube.com/watch?v=xsSnOQynTHs). Его целью было создание библиотеки управления состоянием с минимальным API, но вполне предсказуемым поведением, так чтобы можно было реализовать протоколирование, горячую перезагрузку, путешествия во времени, универсальные приложения, запись и воспроизведение, без каких-либо вложений от разработчика.

### Влияния

Redux развивает идеи [Flux](http://facebook.github.io/flux/), но избегает его сложности, воспользовавшись примерами из [Elm](https://github.com/evancz/elm-architecture-tutorial/).
Вне зависимости от того, использовали вы их или нет, Redux занимает всего несколько минут, чтобы начать с ним работу.

### Установка

Для установки стабильной версии:

```
npm install --save redux
```

Предполагается, что вы используете [npm](https://www.npmjs.com/) в качестве менеджера пакетов.

Если нет, то вы можете [получить доступ к этим файлам на unpkg](https://unpkg.com/redux/), загрузить их или настроить ваш пакетный менеджер.

Чаще всего люди используют Redux, как набор [CommonJS](http://webpack.github.io/docs/commonjs.html) модулей. Эти модули - это то, что вы получаете, когда импортируете `redux` в [Webpack](https://webpack.js.org/), [Browserify](http://browserify.org/) или Node окружение. Если вам нравится жить на острие технологий и использовать [Rollup](http://rollupjs.org), мы также поддерживаем это.

Если вы не используете сборщики модулей, это тоже нормально. Npm пакет `redux` включает предкомпилированные [UMD](https://github.com/umdjs/umd) develop и production сборки в [каталоге `dist`](https://unpkg.com/redux/dist/). Они могут быть использованы напрямую без бандлера и, таким образом, совместимы со многими популярными загрузчиками JavaScript-модулей и окружениями. Например, вы можете подключить UMD-сборку на страницу при помощи [тэга `<script>`](https://unpkg.com/redux/dist/redux.js) или [установить при помощи Bower](https://github.com/reactjs/redux/pull/1181#issuecomment-167361975). Сборки UMD делают Redux доступным через глобальную переменную `window.Redux`.

Исходные коды Redux написаны на ES2015, но мы предкомпилировали и CommonJS и UMD сборки в ES5 поэтому они работают в [любом современном браузере](http://caniuse.com/#feat=es5). Вам нет необходимости использовать Babel или сборщик модулей чтобы [начать пользоваться Redux](https://github.com/reactjs/redux/blob/master/examples/counter-vanilla/index.html).

### Дополнительные пакеты

Скорее всего, вам также понадобится [связка с React](https://github.com/reactjs/react-redux) и [инструменты разработчика](https://github.com/gaearon/redux-devtools).

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

Обратите внимание, что в отличие от самого Redux, многие пакеты в экосистеме Redux не предоставляют сборки UMD, поэтому мы рекомендуем использовать сборщики модулей CommonJS, такие как [Webpack] (https://webpack.js.org/) и [Browserify] (http://browserify.org/) для наиболее комфортной разработки.

### Основные принципы

Все состояние вашего приложения сохранено в объекте внутри одного *хранилища (store)*. 
Единственный способ изменить дерево состояния - это вызвать *действие (action)*, объект описывающий то, что случилось. 
Чтобы указать, каким образом действия преобразовывают дерево состояния, вы пишете чистые "редюсеры".

Все, готово!

```js
import { createStore } from 'redux'

/**
 * Это редюсер - чистая функция в формате (state, action) => state.
 * Он описывает то, как действие преобразовывает состояние в следующее состояние 
 *
 * Формат состояния зависит от вас: это может быть примитивом,
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

// Создаем хранилище Redux которое хранит состояние вашего приложения.
// Его API - { subscribe, dispatch, getState }.
let store = createStore(counter)

// Вы можете использовать subscribe() для обновления UI в ответ на изменения состояния.
// Обычно вы должны использовать библиотеку привязки (например, React Redux), а не использовать subscribe() напрямую.
// Однако также может быть полезно сохранить текущее состояние в localStorage.
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

## Изучайте Redux вместе с его создателем

### Видео уроки Redux от Dan Abramov

#### Getting Started with Redux
**[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)**  - это видео-курс, состоящий из 30 роликов созданных Дэном Абрамовым, автором Redux. Он предназначен для дополнения части документации «Основы», привнося дополнительные сведения о неизменности, тестировании, лучших практиках Redux, а также об использовании Redux с React. **Данный курс является и всегда будет бесплатным.**

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

#### [Посмотрите 30 бесплатных уроков "Getting Started with Redux"!](https://egghead.io/series/getting-started-with-redux)

> Note: Если вам понравился мой курс, подумайте о поддержке Egghead путем [покупки подписки] (https://egghead.io/pricing). Подписчики имеют доступ к исходному коду, для примера, в каждом из моих видео, а также к массе продвинутых уроков по другим темам, включая JavaScript in depth, React, Angular и многое другое. Многие [преподаватели Egghead] (https://egghead.io/instructors) также являются авторами библиотек с открытым исходным кодом, т.ч. покупка подписки - это хороший способ поблагодарить их за работу, которую они сделали.


#### Создание React приложений с помощью Redux

**[Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)** - это вторая бесплатная видео серия от Дэна Абрамова. Он продолжает темы, начатые в первой серии и охватывает практические технологии, необходимые для создания ваших приложений React и Redux: усовершенствованное управление состоянием, мидлвары, интеграция React Router и другие общие проблемы, с которыми вы, вероятно, столкнетесь при создании приложений для своих клиентов и заказчиков. Как и в первой серии, **этот курс всегда будет бесплатным**.


#### [Смотрите бесплатный видео курс "Idiomatic Redux"](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)


### Практический курс Redux

**[Практический Redux](https://www.educative.io/collection/5687753853370368/5707702298738688/)** - платный интерактивный курс одного из главных разработчиков Redux [Mark Erikson](https://twitter.com/acemarke ). Курс предназначен для того, чтобы показать, как применять базовые концепции Redux к созданию чего-то большего, чем приложение TodoMVC. Он включает в себя такие реальные темы, как:


- Добавление Redux к новому проекту Create-React-App и настройка Hot Module Replacement для более быстрой разработки
- Управление вашим пользовательским интерфейсом с помощью Redux
- Использование библиотеки Redux-ORM для управления связанными данными в вашем хранилище Redux
- Создание главного/детального представления для отображения и редактирования данных
- Написание специальной усовершенствованной логики редюсера Redux для решения конкретных задач
- Оптимизация производительности подключенных к Redux полей формы

И многое другое!

Курс основывается на оригинальной бесплатной учебной серии **[«Практический Redux»](http://blog.isquaredsoftware.com/series/practical-redux/)**, но с обновленным и улучшенным контентом.


### Воркшопы Redux

Марк Эриксон - один из разработчиков Redux [Mark Erikson](https://twitter.com/acemarke) совместно с [Workshop.me](https://workshop.me/) проводит серию воркшопов по Redux

Первый семинар [**Redux Fundamentals** состоится в Нью-Йорке, 19-20 апреля](https://workshop.me/2018-04-react-redux?a=mark) и будет знакомить со следующими темами:

- История и цель Redux
- Редюсеры, действия и работа с хранилищем Redux
- Использование Redux с React
- Использование и написание мидлвар для Redux
- Работа с вызовами AJAX и другими побочными эффектами
- Тестирование приложений Redux
- Реальная структура и разработка приложений Redux

Билеты по-прежнему доступны и их [можно приобрести через Workshop.me](https://workshop.me/2018-04-react-redux?a=mark)


## Документация

* [Введение](https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/index.html)
* [Основы](https://rajdee.gitbooks.io/redux-in-russian/content/docs/basics/index.html)
* [Продвинутое использование](https://rajdee.gitbooks.io/redux-in-russian/content/docs/advanced/index.html)
* [Рецепты](https://rajdee.gitbooks.io/redux-in-russian/content/docs/recipes/index.html)
* [Поиск неисправностей](https://rajdee.gitbooks.io/redux-in-russian/content/docs/Troubleshooting.html)
* [Глоссарий](https://rajdee.gitbooks.io/redux-in-russian/content/docs/Glossary.html)
* [Справочник по API](https://rajdee.gitbooks.io/redux-in-russian/content/docs/api/index.html)

Для экпорта в PDF, ePub MOBI или чтения в оффлайн и инструкциям, как это можно осуществить, обратите внимание на: [paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs).

Для документации в Offline, пожалуйста посмотрите: [devdocs](http://devdocs.io/redux/)

## Примеры

Почти все примеры имеют соответствующую песочницу CodeSandbox. Это интерактивная версия кода, с которой вы можете играть онлайн.

* [**Counter Vanilla**](https://redux.js.org/introduction/examples#counter-vanilla): [Source](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla)
* [**Counter**](https://redux.js.org/introduction/examples#counter): [Source](https://github.com/reactjs/redux/tree/master/examples/counter) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/counter)
* [**Todos**](https://redux.js.org/introduction/examples#todos): [Source](https://github.com/reactjs/redux/tree/master/examples/todos) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/todos)
* [**Todos with Undo**](https://redux.js.org/introduction/examples#todos-with-undo): [Source](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/todos-with-undo)
* [**TodoMVC**](https://redux.js.org/introduction/examples#todomvc): [Source](https://github.com/reactjs/redux/tree/master/examples/todomvc) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/todomvc)
* [**Shopping Cart**](https://redux.js.org/introduction/examples#shopping-cart): [Source](https://github.com/reactjs/redux/tree/master/examples/shopping-cart) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/shopping-cart)
* [**Tree View**](https://redux.js.org/introduction/examples#tree-view): [Source](https://github.com/reactjs/redux/tree/master/examples/tree-view) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/tree-view)
* [**Async**](https://redux.js.org/introduction/examples#async): [Source](https://github.com/reactjs/redux/tree/master/examples/async) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/async)
* [**Universal**](https://redux.js.org/introduction/examples#universal): [Source](https://github.com/reactjs/redux/tree/master/examples/universal)
* [**Real World**](https://redux.js.org/introduction/examples#real-world): [Source](https://github.com/reactjs/redux/tree/master/examples/real-world) | [Sandbox](https://codesandbox.io/s/github/reactjs/redux/tree/master/examples/real-world)

Если вы новичок в экосистеме NPM и имеете проблемы с получением и запуском проекта или не уверены, куда вставить шаблон, попробуйте [simplest-redux-example](https://github.com/jackielii/simplest-redux-example) который использует Redux вместе с React и Browserify.

## Отзывы

>[“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen, автор Flux

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher, автор документации Flux

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz, creator of Cycle


## Благодарности

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

## Logo

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

MIT
