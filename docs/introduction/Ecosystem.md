# Экосистема

Redux - это небольшая библиотека, в которой соглашения и API были тщательно продуманы для удобного создания инструментов и подключения расширений экосистемы.

Мы рекомендуем ознакомиться с большим списком репозиториев, связанных с Redux в [Awesome Redux](https://github.com/xgrommx/awesome-redux).
Он содержит примеры, макеты, промежуточное ПО (middleware), библиотеки утилит и т.д. 

На этой странице мы приводим только часть из данного списка репозиториев, функционал которых был лично проверен разработчиками Redux. Однако, пусть это не ограничивает вас в экспериментах со сторонними модулями. Экосистема Redux растет очень быстро, но нам не хватает времени, чтобы отслеживать все изменения.

Итак, ознакомьтесь с нашим "избранным" и не стесняйтесь делиться своими наработками, если вы сделали что-то удивительное с Redux.

## Связь с DOM (Bindings)

* [react-redux](https://github.com/gaearon/react-redux) — React
* [ng-redux](https://github.com/wbuchwalter/ng-redux) — Angular
* [ng2-redux](https://github.com/wbuchwalter/ng2-redux) — Angular 2
* [backbone-redux](https://github.com/redbooth/backbone-redux) — Backbone
* [redux-falcor](https://github.com/ekosz/redux-falcor) — Falcor
* [deku-redux](https://github.com/troch/deku-redux) — Deku (6кб альтернатива для React)

## Промежуточное ПО (Middleware)

* [redux-thunk](http://github.com/gaearon/redux-thunk) — Самый легкий способ создавать асинхронные генераторы действий (action creators) 
* [redux-promise](https://github.com/acdlite/redux-promise) — [FSA](https://github.com/acdlite/flux-standard-action)-совместимый promise middleware
* [redux-rx](https://github.com/acdlite/redux-rx) — RxJS утилиты для Redux, включая  middleware для Observable
* [redux-logger](https://github.com/fcomb/redux-logger) — Логирование каждого Redux действия (action) и следующего состояния (state)
* [redux-immutable-state-invariant](https://github.com/leoasis/redux-immutable-state-invariant) — Предупреждения об изменениях состояния во время разработки
* [redux-analytics](https://github.com/markdalgleish/redux-analytics) — Аналитика для Redux

## Компоненты (Components)

* [redux-form](https://github.com/erikras/redux-form) — Поддержка Redux состояний (stores) для html форм, использующихся в React

## Улучшения Store

* [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) — Настройка групповых и отложенных вызовов для подписавшихся на store
* [redux-history-transitions](https://github.com/johanneslumpe/redux-history-transitions) — переходы по History основанные на произвольных действиях (actions).

## Улучшения Reducer

* [redux-optimist](https://github.com/ForbesLindesay/redux-optimist) — Оптимистичное использование действий (action) с возможностью их совершения или отмены в дальнейшем. 
> От переводчика: например, можно инициировать `ADD_TODO` action, state изменится, UI обновится мгновенно, а затем отправится запрос на сохранение на сервер. Если на сервере все прошло успешно, то все ок и больше ничего не произойдет. Но если же что-то пошло не так, то state будет "отмотан" до состояния когда `ADD_TODO` action еще не был применен, state изменится, UI снова обновится мгновенно. Т.е. UI будет выглядеть так, словно ничего и не произошло.
* [redux-undo](https://github.com/omnidan/redux-undo) — Позволяет без усилий получить undo/redo функциональность и историю действий для редюсеров
    * [redux-ignore](https://github.com/omnidan/redux-ignore) — Ignore redux actions by array or filter function
* [redux-recycle](https://github.com/omnidan/redux-recycle) — Reset the redux state on certain actions
* [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) — Dispatch several actions with a single subscriber notification

## Утилиты (Utilities)

* [reselect](https://github.com/faassen/reselect) — Простая библиотека "селекторов" нашедшая вдохновение в геттерах NuclearJS
* [normalizr](https://github.com/gaearon/normalizr) — Упорядочивание вложенных ответов API для облегчения дальнейшего их использования в редюсерах
* [redux-actions](https://github.com/acdlite/redux-actions) — Уменьшение шаблонности в написании редюсеров и генераторов действий (action creators)
* [redux-transducers](https://github.com/acdlite/redux-transducers) — Transducer утилиты для Redux
* [redux-immutablejs](https://github.com/indexiatech/redux-immutablejs) — Инструменты интеграции между Redux и [Immutable](https://github.com/facebook/immutable-js/)
* [redux-tcomb](https://github.com/gcanti/redux-tcomb) — Иммутабельные, с проверкой типов состояния и действия для Redux


## Инструменты разработчика (Developer Tools)

* [Redux DevTools](http://github.com/gaearon/redux-devtools) — Логирование действий с UI для путешествий во времени, горячая перезагрузка и обработка ошибок для редюсеров, [впервые представлено на React Europe](https://www.youtube.com/watch?v=xsSnOQynTHs)
* [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension) - Плагин для Chrome, по сути обертка для Redux DevTools с дополнительной функциональностью

## Туториалы и статьи (Tutorials and Articles)

* [redux-tutorial](https://github.com/happypoulp/redux-tutorial) — Обучающее HowTo для Redux. Шаг за шагом.
* [What the Flux?! Let’s Redux.](https://blog.andyet.com/2015/08/06/what-the-flux-lets-redux) — Введение в Redux
* [Handcrafting an Isomorphic Redux Application (With Love)](https://medium.com/@bananaoomarang/handcrafting-an-isomorphic-redux-application-with-love-40ada4468af4) — Руководство, рассказывающее как создать универсальное приложение с фетчингом и роутингом
* [Full-Stack Redux Tutorial](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html) — Всестороннее руководство по test-first разработке с Redux, React и Immutable

## Выступления (Talks)

* [Live React: Hot Reloading and Time Travel](http://youtube.com/watch?v=xsSnOQynTHs) — Какие ограничения, диктуемые Redux, делают легкими перезагрузку с путешествиями во времени
* [Cleaning the Tar: Using React within the Firefox Developer Tools](https://www.youtube.com/watch?v=qUlRpybs7_c) — Как постепенно перевести на Redux существующее MVC приложение

## Общественные соглашения (Community Conventions)

* [Flux Standard Action](https://github.com/acdlite/flux-standard-action) — Дружелюбный стандарт для Flux action объектов
* [Canonical Reducer Composition](https://github.com/gajus/canonical-reducer-composition) — Слишком самоуверенный (opinionated) стандарт для структуры вложенных редюсеров
* [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) — Предложение по связыванию редюсеров, типов действий и действий

## Прочее (More)

[Awesome Redux](https://github.com/xgrommx/awesome-redux) — обширный список репозиториев, имеющих отношение к Redux.
