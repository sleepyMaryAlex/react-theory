# ?Redux

### Enumerate base principles

Redux можно описать тремя фундаментальными принципами:

* Единственный источник правды. Глобальное состояние вашего приложения хранится в одном хранилище.
* Состояние доступно только для чтения. Единственный способ изменить `state` — запустить действие, объект, описывающий, что произошло.
* Изменения вносятся с помощью чистых функций. Чтобы указать, как `state` преобразуется действиями, вы пишете чистые `reducers`.

### What is the typical flow of data in a React + Redux app?

Redux использует структуру приложения «односторонний поток данных».

Вот как этот поток данных выглядит визуально:

![Redux data flow](../images/redux-data-flow.gif)

* Состояние описывает состояние приложения в определенный момент времени.
* Пользовательский интерфейс отображается на основе этого состояния
* Когда что-то происходит (например, когда пользователь нажимает кнопку), состояние обновляется в зависимости от того, что произошло.
* Пользовательский интерфейс перерисовывается на основе нового состояния

`action` — это простой объект JavaScript с полем `type`. Вы можете думать о `action` как о событии, описывающем что-то, что произошло в приложении. Объект действия может иметь другие поля с дополнительной информацией о том, что произошло. По соглашению мы помещаем эту информацию в поле с именем `payload`.

~~~
const addTodoAction = {
  type: 'todos/todoAdded',
  payload: 'Buy milk'
}
~~~

`reducer` — это функция, которая получает текущий `state` и объект `action`, решает, как обновить состояние при необходимости, и возвращает новое состояние.

`Reducers` всегда должны следовать определенным правилам:

* Они должны только вычислить новое значение состояния на основе аргументов `state` и `action`.
* Им не разрешено изменять существующий `state`. Вместо этого они должны делать неизменяемые обновления, копируя существующие `state` и внося изменения в скопированные значения.
* Они не должны выполнять асинхронную логику, вычислять случайные значения или вызывать другие «побочные эффекты».

~~~
function counterReducer(state = initialState, action) {
  if (action.type === 'counter/incremented') {
    return { ...state, value: state.value + 1 };
  }
  return state;
}
~~~

Текущее состояние приложения Redux находится в объекте, называемом `store`.

Единственный способ обновить состояние — вызвать `store.dispatch()` и передать объект `action`.

### Benefits of Redux

* Простая передача состояния между компонентами.
* Предсказуемые состояния.
* Простота в обслуживании. Redux строго следит за тем, как должен быть организован код, поэтому для тех, кто знает Redux, проще понять структуру любого приложения.
* Тестировать код просто.
* Доступны обширные инструменты разработчика.
* Большое поддерживающее сообщество.
* Чистые функции.