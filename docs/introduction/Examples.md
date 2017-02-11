# Примеры

Redux распространяется с несколькими примерами в своих [исходных кодах](https://github.com/reactjs/redux/tree/master/examples).
**Для того, чтобы запустить любой из них, достаточно склонировать репозиторий и запустить `npm install` в родительской директории и в директории примера.**

>##### Особенности при копировании
>Если вы скопировали примеры Redux из их директорий, удалите эти строки из файлов `webpack.config.js`:
>
>```js
>alias: {
>   'redux': path.join(__dirname, '..', '..', 'src')
>},
>```
>и
>```js
>{
>   test: /\.js$/,
>   loaders: ['babel'],
>   include: path.join(__dirname, '..', '..', 'src')
>},
```
>
> Иначе они будут пытаться вызвать Redux из родительской `src` папки и сборка сломается.

## Счетчик

Запуск примера [счетчика](https://github.com/reactjs/redux/tree/master/examples/counter):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/counter
npm install
npm start

open http://localhost:3000/
```

Тут показаны:

* Базовый шаблон разработки Redux;
* Тестирование.

## TodoMVC

Запуск [TodoMVC](https://github.com/reactjs/redux/tree/master/examples/todomvc):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/todomvc
npm install
npm start

open http://localhost:3000/
```

Тут показаны:

* Базовый шаблон разработки Redux с двумя редюсерами;
* Обновление хранимой информации;
* Тестирование.

## Async

Запуск [Async](https://github.com/reactjs/redux/tree/master/examples/async):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/async
npm install
npm start

open http://localhost:3000/
```

Тут показаны:

* Базовый шаблон асинхронной разработки Redux с [redux-thunk](https://github.com/gaearon/redux-thunk);
* Кеширование запросов и вывод спиннера, пока данные подгружаются;
* Инвалидация закэшированных данных.

## Real World

Запуск примера [Real World](https://github.com/reactjs/redux/tree/master/examples/real-world):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/real-world
npm install
npm start

open http://localhost:3000/
```

Тут показаны:

* Real-world пример асинхронной работы с Redux;
* Сохранение объектов в упорядоченном кэше объектов;
* Отдельный middleware для вызовов API;
* Кеширование запросов и вывод спиннера, пока данные подгружаются;
* Пагинация;
* Роутинг.

## Shopping Cart

Запуск примера [Shopping Cart](https://github.com/reactjs/redux/tree/master/examples/shopping-cart):

```
git clone https://github.com/gaearon/redux.git

cd redux
npm install

cd examples/shopping-cart
npm install
npm start

open http://localhost:3000/
```

Тут показаны:

* Нормализованное состояние
* Явное отслеживание ID сущностей
* Компоновка редюсеров
* Использование очередей наряду с редюсерами
* Откат при ошибке
* Безопасное распространение действий по условию
* Использование только React Redux для привязки генераторов действий
* Условные мидлвэры (на примере логирования)

## Больше примеров

Больше примеров вы сможете найти в [Awesome Redux](https://github.com/xgrommx/awesome-redux).
