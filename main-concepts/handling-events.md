# ?Handling events

~~~
<button onClick={activateLasers}>
  Активировать лазеры
</button
~~~

При использовании React обычно не нужно вызывать `addEventListener`, чтобы добавить обработчики в DOM-элемент после его создания. Вместо этого добавьте обработчик сразу после того, как элемент отрендерился.

В компоненте, определённом с помощью ES6-класса, в качестве обработчика события обычно выступает один из методов класса.

~~~
class Toggle extends React.Component<object, { isToggleOn: boolean }> {
  constructor(props: object) {
    super(props);
    this.state = {isToggleOn: true};
    // Эта привязка обязательна для работы `this` в колбэке.
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'Включено' : 'Выключено'}
      </button>
    );
  }
}

export default Toggle;
~~~

При обращении к `this` в JSX-колбэках необходимо учитывать, что методы класса в JavaScript по умолчанию не привязаны к контексту. Если вы забудете привязать метод `this.handleClick` и передать его в `onClick`, значение `this` будет `undefined` в момент вызова функции.

Существует два других способа привязать метод:

* Сделать `handleClick` стрелочной функцией. Стрелочная функция не имеет своего контекста выполнения. `this` в стрелочной функции, как обычная переменная, берётся из внешнего лексического окружения.
* Стрелочная функция в `callback`:

~~~
onClick={() => this.handleClick()}>
~~~

Второй способ не самый лучший. Рекомендуется делать привязку в конструкторе или использовать синтаксис полей классов, чтобы избежать проблем с производительностью.

### Passing Arguments to Event Handlers

Внутри цикла часто нужно передать дополнительный аргумент в обработчик события. Например, если `id` — это идентификатор строки, можно использовать следующие варианты:

~~~
<button onClick={(e) => this.deleteRow(id, e)}>Удалить строку</button>
<button onClick={this.deleteRow.bind(this, id)}>Удалить строку</button>
~~~

Две строки выше — эквивалентны.

Используя стрелочную функцию, необходимо передавать аргумент `e` явно, но с `bind` `e` передается автоматически.
