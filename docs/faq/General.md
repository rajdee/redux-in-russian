# Redux FAQ: Главное

## Содержание

- [Когда я должен изучать Redux?](#when-should-i-learn-redux)
- [Когда я должен использовать Redux?](#general-when-to-use) 
- [Должен ли Redux быть использован только с React?](#general-only-react) 
- [Нужны ли мне дополнительные инструменты для использования Redux?](#general-build-tools) 

## Главное

<a id="when-should-i-learn-redux"></a>

### Когда я должен изучать Redux?

Что изучать, может быть непреодолимым вопросом для разработчика JavaScript. Это помогает сузить диапазон вариантов, изучая одну вещь за раз и концентрируясь на проблемах, которые вы обнаружите в своей работе. Redux - это шаблон для управления состоянием приложения. Если у вас нет проблем с управлением состоянием, вам может быть труднее понять преимущества Redux. Некоторые библиотеки пользовательского интерфейса (например, React) имеют собственную систему управления состоянием. Если вы используете одну из этих библиотек, особенно если вы только учитесь их использовать, мы рекомендуем вам сначала изучить возможности этой встроенной системы. Это может быть все, что вам нужно для создания приложения. Если ваше приложение становится настолько сложным, что вы не понимаете, где хранится состояние или как оно изменяется, тогда самое время изучить Redux. Испытывание сложности, которую Redux стремится абстрагировать, является наилучшей подготовкой для эффективного применения этой абстракции к вашей работе.

#### Дальнейшая информация

**Статьи**

- [Deciding What Not To Learn](http://gedd.ski/post/what-not-to-learn/)
- [How to learn web frameworks](https://ux.shopify.com/how-to-learn-web-frameworks-9d447cb71e68)
- [Redux vs MobX vs Flux vs... Do you even need that?](https://goshakkk.name/redux-vs-mobx-vs-flux-etoomanychoices/)

**Обсуждения**

- [Ask HN: Overwhelmed with learning front-end, how do I proceed?](https://news.ycombinator.com/item?id=12882816)
- [Twitter: If you want to teach someone to use an abstraction...](https://twitter.com/acemarke/status/901329101088215044)
- [Twitter: it was never intended to be learned before...](https://twitter.com/dan_abramov/status/739961787295117312)
- [Twitter: Learning Redux before React?](https://twitter.com/dan_abramov/status/739962098030137344)
- [Twitter: The first time I used React, people told me I needed Redux...](https://twitter.com/raquelxmoss/status/901576285020856320)
- [Twitter: This was my experience with Redux...](https://twitter.com/garetmckinley/status/901500556568645634)
- [Dev.to: When is it time to use Redux?](https://dev.to/dan_abramov/comment/1n2k)


<a id="general-when-to-use"></a>

### Когда я должен использовать Redux? 

Необходимость использовать Redux не следует воспринимать как должное.

Пит Хант (Pete Hunt), один из первых контрибьюторов React говорит:

> Вы поймете, когда вам нужен Flux. Если вы не уверены, что вам это нужно, вам это не нужно.

В тоже время, Дэн Абрамов (Dan Abramov), один из создателей Redux, говорит:

> Я хотел бы уточнить: не используйте Redux, пока у вас есть проблемы с "ванильным" React.

В общем, используйте Redux, когда у вас есть обоснованные объемы данных, изменяющихся с течением времени, когда вам нужен один источник информации и когда вы обнаружите, что подходов, типа хранения всего в стейте высокоуровненого React-компонента, уже недостаточно.

Тем не менее, также важно понимать, что использование Redux сопряжено с компромиссами. Он не создан чтобы быть самым коротким или самым быстрым способом написания кода.
Он предназначен для того, чтобы помочь ответить на вопрос - "Когда же изменилась определенная часть состояния или откуда эти данные" с предсказуемым поведением. Он делает это, прося вас следовать определенным требованиям в вашем приложении:
хранить состояние вашего приложения в виде простой структуры данных, описывать изменения в виде простых объектов и обрабатывать эти изменения с помощью чистых функций, которые применяют изменения иммутабельно. Это часто является источником жалоб на "шаблонность". Эти ограничения требуют усилий со стороны разработчика, но также и предоставляют ряд дополнительных возможностей (например, сохранение стора и синхронизацию).

В конце концов, Redux - это всего лишь инструмент. Это отличный инструмент и есть ряд отличных причин, чтобы использовать его, но есть также причины, когды вы можете не захотеть использовать его.
Принимайте обоснованные решения о своих инструментах и понимайте компромиссы, участвующих в каждом решении.

#### Дополнительная информация

**Документация**
- [Introduction: Motivation](/docs/introduction/Motivation.md)

**Статьи**

- [React How-To](https://github.com/petehunt/react-howto)
- [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
- [The Case for Flux](https://medium.com/swlh/the-case-for-flux-379b7d1982c6)
- [Some Reasons Why Redux is Useful in a React App](https://www.fullstackreact.com/articles/redux-with-mark-erikson/)

**Обсуждения**

- [Twitter: Don't use Redux until...](https://twitter.com/dan_abramov/status/699241546248536064)
- [Twitter: Redux is designed to be predictable, not concise](https://twitter.com/dan_abramov/status/733742952657342464)
- [Twitter: Redux is useful to eliminate deep prop passing](https://twitter.com/dan_abramov/status/732912085840089088)
- [Twitter: Don't use Redux unless you're unhappy with local component state](https://twitter.com/dan_abramov/status/725089243836588032)
- [Twitter: You don't need Redux if your data never changes](https://twitter.com/dan_abramov/status/737036433215610880)
- [Twitter: If your reducer looks boring, don't use redux](https://twitter.com/dan_abramov/status/802564042648944642)
- [Reddit: You don't need Redux if your app just fetches something on a single page](https://www.reddit.com/r/reactjs/comments/5exfea/feedback_on_my_first_redux_app/dagglqp/)
- [Stack Overflow: Why use Redux over Facebook Flux?](http://stackoverflow.com/questions/32461229/why-use-redux-over-facebook-flux)
- [Stack Overflow: Why should I use Redux in this example?](http://stackoverflow.com/questions/35675339/why-should-i-use-redux-in-this-example)
- [Stack Overflow: What could be the downsides of using Redux instead of Flux?](http://stackoverflow.com/questions/32021763/what-could-be-the-downsides-of-using-redux-instead-of-flux)
- [Stack Overflow: When should I add Redux to a React app?](http://stackoverflow.com/questions/36631761/when-should-i-add-redux-to-a-react-app)
- [Stack Overflow: Redux vs plain React?](http://stackoverflow.com/questions/39260769/redux-vs-plain-react/39261546#39261546)
- [Twitter: Redux is a platform for developers to build customized state management with reusable things](https://twitter.com/acemarke/status/793862722253447168)

<a id="general-only-react"></a>

### Должен ли Redux быть использован только с React?

Redux может быть использован в качестве хранилища данных с любым уровнем представления. Чаще всего он используется с React и React Native, но также доступны биндинги для Angular, Angular 2, Vue, Mithril и других. Redux просто предоставляет механизм подписки, который может быть использован любым другим кодом. Тем не менее, это наиболее полезно в сочетании с декларативной реализацией представления, которая может иметь зависимость обновления пользовательского интерфейса от изменений состояния, такие как React или одна из подобных библиотек.

<a id="general-build-tools"></a>
### Нужны ли мне дополнительные инструменты для использования Redux?

Redux изначально написан на ES6 и транспилирован в ES5 при помощи Webpack и Babel. Вы должны быть в состоянии использовать его независимо от процесса сборки JavaScript. Redux также предоставляет UMD-сборку, которую можно использовать непосредственно без какого-либо процесса сборки вообще. Пример [counter-vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) демонстрирует основы использования ES5 с Redux, подключенного при помощи тега `<script>`. Как соответствующий pull request говорит:

> Новый пример Counter Vanilla призван развеять миф о том, что Redux`у необходим Webpack, React, hot reloading, саги, генераторы экшенов, константы, Babel, npm, CSS модули, декораторы, беглый латинский, подписка на Egghead, степень доктора философии или уровень "Выше Ожидаемого" СОВ.

>Неа, это всего лишь HTML, несколько кустарных `<script>` тэгов и старые добрые манипуляции с DOM. Наслаждайтесь!
