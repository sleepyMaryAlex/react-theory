# ?Forms

В React HTML-элементы формы ведут себя немного иначе по сравнению с DOM-элементами, так как у элементов формы изначально есть внутреннее состояние.

Чаще всего форму удобнее обрабатывать с помощью JavaScript-функции, у которой есть доступ к введённым данным. Стандартный способ реализации такого поведения называется «управляемые компоненты».

В React мутабельное состояние обычно содержится в свойстве компонентов `state` и обновляется только через вызов `setState()`.

### Тег `input`

Допустим, мы хотим, чтобы следующий пример показал в модальном окне введённое имя, когда мы отправляем форму. Тогда можно написать форму в виде управляемого компонента:

~~~
class NameForm extends React.Component<object, { value: string }> {
  constructor(props: object) {
    super(props);
    this.state = { value: '' };
  }

  handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    this.setState({value: (event.target as HTMLInputElement).value});
  }

  handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    alert('Отправленное имя: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit.bind(this)}>
        <label>
          Имя:
          <input type="text" value={this.state.value} onChange={this.handleChange.bind(this)} />
        </label>
        <input type="submit" value="Отправить" />
      </form>
    );
  }
}
~~~

Мы установили атрибут `value` для поля ввода и теперь в нём всегда будет отображаться значение `this.state.value`. Состояние React-компонента стало «источником истины». А так как каждое нажатие клавиши вызывает `handleChange`, который обновляет состояние React-компонента, значение в поле будет обновляться по мере того, как пользователь печатает.

В управляемом компоненте значение поля ввода всегда определяется состоянием React. Хотя это означает, что вы должны написать немного больше кода, теперь вы сможете передать значение и другим UI-элементам или сбросить его с других обработчиков событий.

### Тег `textarea`

В React `<textarea>` использует атрибут `value`. Таким образом, форму с `<textarea>` можно написать почти тем же способом, что и форму с однострочным `<input>`:

~~~
class EssayForm extends React.Component<object, {value: string}> {
  constructor(props: object) {
    super(props);
    this.state = {
      value: 'Write something'
    };
  }

  handleChange(event: React.ChangeEvent<HTMLTextAreaElement>) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    alert('Сочинение отправлено: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit.bind(this)}>
        <label>
          Сочинение:
          <textarea value={this.state.value} onChange={this.handleChange.bind(this)} />
        </label>
        <input type="submit" value="Отправить" />
      </form>
    );
  }
}
~~~

### Тег `select`

В HTML пункт списка `coconut` выбран по умолчанию из-за установленного атрибута `selected`. React вместо этого атрибута использует `value` в корневом теге `select`. В управляемом компоненте так удобнее, потому что обновлять значение нужно только в одном месте (`state`). Пример:

~~~
class EssayForm extends React.Component<object, {value: string}> {
  constructor(props: object) {
    super(props);
    this.state = {
      value: 'coconut'
    };
  }

  handleChange(event: React.ChangeEvent<HTMLSelectElement>) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    alert('Your favorite taste: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit.bind(this)}>
        <label>
          Choose your favorite taste:
          <select value={this.state.value} onChange={this.handleChange.bind(this)}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Send" />
      </form>
    );
  }
}
~~~

В атрибут `value` можно передать массив, что позволит выбрать несколько опций в теге `select`:

~~~
<select multiple={true} value={['Б', 'В']}></select>
~~~

Подводя итог, `<input type="text">`, `<textarea>`, и `<select>` работают очень похоже. Все они принимают атрибут `value`, который можно использовать, чтобы реализовать управляемый компонент.

### `input type="file"`

В HTML `<input type="file">` позволяет пользователю выбрать один или несколько файлов для загрузки. Так как значение такого элемента доступно только для чтения, это неуправляемый React-компонент.

### Handling multiple input elements

Если вам нужны несколько управляемых элементов `input`, вы можете назначить каждому из них атрибут `name`, что позволит функции-обработчику решать, что делать, основываясь на значении `event.target.name`.

~~~
class Reservation extends React.Component<object, { [x: string]: number | boolean }> {
  constructor(props: object) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };
  }

  handleInputChange(event: React.ChangeEvent<HTMLInputElement>) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;
    this.setState({[name]: value as number | boolean});
  }

  render() {
    return (
      <form>
        <label>
          Пойдут:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing as boolean}
            onChange={this.handleInputChange.bind(this)} />
        </label>
        <br />
        <label>
          Количество гостей:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests as number}
            onChange={this.handleInputChange.bind(this)} />
        </label>
      </form>
    );
  }
}
~~~

Если установить управляемому компоненту проп `value`, то пользователь не сможет изменить его значение без вашего желания. Если вы установили `value`, а поле ввода по-прежнему можно редактировать, то, возможно, вы случайно задали `value`, равный `undefined` или `null`.
