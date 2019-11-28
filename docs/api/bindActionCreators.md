# `bindActionCreators(actionCreators, dispatch)`

Преобразует объект, значениями которого являются [генераторы экшенов](../Glossary.md#action-creator), в объект с теми же ключами, но генераторами экшенов, обернутыми в вызов [`dispatch`](Store.md#dispatch), т.ч. они могут быть вызваны напрямую.

Как правило, вы просто должны вызвать [`dispatch`](Store.md#dispatch) прямо в вашем инстансе [`Store`](Store.md). Если вы используете Redux c React, то [react-redux](https://github.com/gaearon/react-redux) предоставит вам [`dispatch`](Store.md#dispatch) функцию, т.ч. вы также сможете вызвать его напрямую.

Единственный случай использования для `bindActionCreators` - это когда вы хотите передать некоторые генераторы экшенов (action creators) вниз к компоненту, который ничего не знает о Redux и вы не хотите передавать ему [`dispatch`](Store.md#dispatch) или Redux стор.

Для удобства, вы также можете передать одну функцию в качестве первого аргумента и получить функцию в ответ.

#### Параметры

1. `actionCreators` (*Функция* или *Объект*): [Генератор экшена](../Glossary.md#action-creator) или объект, значениями которого являются генераторы экшенов.

2. `dispatch` (*Функция*): [`dispatch`](Store.md#dispatch) функция доступная в инстансе [`Store`](Store.md).

#### Возвращает

(*Функция* или *Объект*): Объект повторяющий исходный объект, но с функциями непосредственно отправляющими экшен (action), возвращаемый соответствующим генератором экшенов. Если вы передаете единственную функцию, как `actionCreators`, возвращаемое значение также будет единственной функцией.

#### Пример

#### `TodoActionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

#### `SomeComponent.js`

```js
import { Component } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import * as TodoActionCreators from './TodoActionCreators'
console.log(TodoActionCreators)
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  constructor(props) { 
    super(props);
      
    const {dispatch} = props;

    // Это хороший случай использования для bindActionCreators:
    // Вы хотите, чтобы дочерний компонент, ничего не знал о Redux.
    // Теперь мы создаем связанные версии этих функций, чтобы мы могли
    // позже передать их нашему потомку.

    this.boundActionCreators = bindActionCreators(TodoActionCreators, dispatch)
    console.log(this.boundActionCreators)
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }
  }

  componentDidMount() {
    // Injected by react-redux:
    let { dispatch } = this.props

    // Примечание: так не будет работать:
    // TodoActionCreators.addTodo('Use Redux')

    // Вы просто вызываете функцию, которая создает экшен.
    // Вы также должны диспатчнуть экшен (action)!

    // Так будет работать:
    let action = TodoActionCreators.addTodo('Use Redux')
    dispatch(action)
  }

  render() {
    // Injected by react-redux:
    let { todos } = this.props

    return <TodoList todos={todos} {...this.boundActionCreators} />

    // Альтернативой для bindActionCreators может быть передача вниз
    // только dispatch функции, но тогда ваш дочерний компонент
    // должен импортировать генераторы экшенов и знать о них.

    // return <TodoList todos={todos} dispatch={dispatch} />
  }
}

export default connect(
  state => ({ todos: state.todos })
)(TodoListContainer)
```

#### Советы

* Вы можете спросить: почему мы не привязываем генераторы экшенов сразу к инстансу стора, как в классическом Flux?  Проблема в том, что это не будет хорошо работать с универсальными приложениями, которые необходимо рендерить на сервере. Скорее всего, вы хотите иметь отдельный инстанс стора для каждого запроса, чтобы вы могли подготовить их с различными данными, но связывание генераторов экшенов во время их определения, означает, что вы привязаны к одному инстансу стора для всех запросов.

* Если вы используете ES5, вместо синтаксиса `import * ` вы можете просто передать `require('./TodoActionCreators')` в `bindActionCreators` в качестве первого аргумента. Единственное, что его волнует, чтобы значения аргументов `actionCreators` были функциями. Модульная система не имеет значения.
