# Redux FAQ: Главное

## Содержание

- [Когда я должен использовать Redux?](#general-when-to-use) 
- [Должен ли Redux быть использован только с React?](#general-only-react) 
- [Нужны ли мне дополнительные инструменты для использования Redux?](#general-build-tools) 


## Главное

<a id="general-when-to-use"></a>
### Когда я должен использовать Redux? 

Пит Хант (Pete Hunt), один из первых контрибьюторов React говорит:

> Вы поймете, когда вам нужен Flux. Если вы не уверены, что вам это нужно, вам это не нужно.

В тоже время, Дэн Абрамов (Dan Abramov), один из создателей Redux, говорит:

> Я хотел бы уточнить: не используйте Redux, пока у вас есть проблемы с "ванильным" React.

В общем, используйте Redux, когда у вас есть обоснованные объемы данных, изменяющихся с течением времени, когда вам нужен один источник информации и когда вы обнаружите, что подходов, типа хранения всего в стейте высокоуровненого React-компонента, уже недостаточно.

Тем не менее, также важно понимать, что использование Redux сопряжено с компромиссами. Он не создан чтобы быть самым коротким или самым быстрым способом написания кода.
Он предназначен для того, чтобы помочь ответить на вопрос - "Когда же изменилась определенная часть состояния или откуда эти данные" с предсказуемым поведением. Он делает это, прося вас следовать определенным требованиям в вашем приложении:
хранить состояние вашего приложения в виде простой структуры данных, описывать изменения в виде простых объектов и обрабатывать эти изменения с помощью чистых функций, которые применяют изменения иммутабельно. Это часто является источником жалоб на "шаблонность". Эти ограничения требуют усилий со стороны разработчика, но также и предоставляют ряд дополнительных возможностей (например, сохранение стора и синхронизацию).

Если вы пока только изучаете React, вы вероятно должны, в первую очередь, сфокусироваться на мышлении в рамках React, а затем посмотреть на Redux, как только вы лучше поймете React и как Redux может вписаться в ваше приложение.

В конце концов, Redux - это всего лишь инструмент. Это отличный инструмент и есть ряд отличных причин, чтобы использовать его, но есть также причины, когды вы можете не захотеть использовать его.
Принимайте обоснованные решения о своих инструментах и понимайте компромиссы, участвующих в каждом решении.

#### Дополнительная информация

**Документация**
- [Introduction: Motivation](/docs/introduction/Motivation.md)

**Статьи**

- [React How-To](https://github.com/petehunt/react-howto)
- [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
- [The Case for Flux](https://medium.com/swlh/the-case-for-flux-379b7d1982c6)

**Обсуждения**

- [Twitter: Don't use Redux until...](https://twitter.com/dan_abramov/status/699241546248536064)
- [Twitter: Redux is designed to be predictable, not concise](https://twitter.com/dan_abramov/status/733742952657342464)
- [Twitter: Redux is useful to eliminate deep prop passing](https://twitter.com/dan_abramov/status/732912085840089088)
- [Twitter: Don't use Redux unless you're unhappy with local component state](https://twitter.com/dan_abramov/status/725089243836588032)
- [Twitter: You don't need Redux if your data never changes](https://twitter.com/dan_abramov/status/737036433215610880)
- [Stack Overflow: Why use Redux over Facebook Flux?](http://stackoverflow.com/questions/32461229/why-use-redux-over-facebook-flux)
- [Stack Overflow: Why should I use Redux in this example?](http://stackoverflow.com/questions/35675339/why-should-i-use-redux-in-this-example)
- [Stack Overflow: What could be the downsides of using Redux instead of Flux?](http://stackoverflow.com/questions/32021763/what-could-be-the-downsides-of-using-redux-instead-of-flux)
- [Stack Overflow: When should I add Redux to a React app?](http://stackoverflow.com/questions/36631761/when-should-i-add-redux-to-a-react-app)


<a id="general-only-react"></a>
### Должен ли Redux быть использован только с React?

Redux может быть использован в качестве хранилища данных с любым уровнем представления. Чаще всего он используется с React и React Native, но также доступны биндинги для Angular, Angular 2, Vue, Mithril и других. Redux просто предоставляет механизм подписки, который может быть использован любым другим кодом. Тем не менее, это наиболее полезно в сочетании с декларативной реализацией представления, которая может иметь зависимость обновления пользовательского интерфейса от изменений состояния, такие как React или одна из подобных библиотек.

<a id="general-build-tools"></a>
### Нужны ли мне дополнительные инструменты для использования Redux?

Redux изначально написан на ES6 и транспилирован в ES5 при помощи Webpack и Babel. Вы должны быть в состоянии использовать его независимо от процесса сборки JavaScript. Redux также предоставляет UMD сборку, которую можно использовать непосредственно без какого-либо процесса сборки вообще. Пример [counter-vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) демонстрирует основы использования ES5 с Redux, подключенного при помощи тега `<script>`. Как соответствующий pull request говорит:

> Новый пример Counter Vanilla призван развеять миф о том, что Redux`у необходим Webpack, React, hot reloading, саги, генераторы действий, константы, Babel, npm, CSS модули, декораторы, беглый латинский, подписка на Egghead, степень доктора философии или уровень "Выше Ожидаемого" СОВ.

>Неа, это всего лишь HTML, несколько кустарных `<script>` тэгов и старые добрые манипуляции с DOM. Наслаждайтесь!
