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

### Почему стрелочные функции и bind в React Render проблематичны

Это делает `shouldComponentUpdate` и `PureComponent` капризными.

App.tsx
~~~
class App extends React.Component<
  {},
  { users: { id: number; name: string }[] }
> {
  constructor(props: object) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: "Cory" },
        { id: 2, name: "Meg" },
        { id: 3, name: "Bob" },
      ],
    };
  }

  deleteUser(id: number) {
    this.setState((prevState) => {
      return {
        users: prevState.users.filter((user) => user.id !== id),
      };
    });
  };

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map((user) => {
            return (
              <User
                key={user.id}
                name={user.name}
                onDeleteClick={() => this.deleteUser(user.id)}
              />
            );
          })}
        </ul>
      </div>
    );
  }
}

export default App;
~~~

User.tsx
~~~
class User extends React.PureComponent<{
  name: string;
  onDeleteClick: () => void;
}> {
  render() {
    const { name, onDeleteClick } = this.props;
    console.log(`${name} just rendered`);
    return (
      <li>
        <input type="button" value="Delete" onClick={onDeleteClick} />
        {name}
      </li>
    );
  }
}

export default User;
~~~

Здесь `User` объявлен как `PureComponent`. Таким образом, пользователь должен повторно отображать только при изменении пропсов или состояния. Но когда вы нажимаете «Удалить» пользователя, обратите внимание, что рендеринг вызывается для всех экземпляров `User`.

И вот почему: родительский компонент передает стрелочную функцию. Стрелочные функции перераспределяются при каждом рендере (та же история с использованием привязки). Поэтому, хоть `User.js` объявлен как `PureComponent`, стрелочная функция в родительском элементе заставляет компонент `User` видеть новую функцию, отправляемую в свойствах для всех пользователей. Таким образом, каждый пользователь выполняет повторный рендеринг при нажатии любой кнопки удаления.

Как исправить:

App.tsx
~~~
class App extends React.Component<
  {},
  { users: { id: number; name: string }[] }
> {
  constructor(props: object) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: "Cory" },
        { id: 2, name: "Meg" },
        { id: 3, name: "Bob" },
      ],
    };
    this.deleteUser = this.deleteUser.bind(this);
  }

  deleteUser(id: number) {
    this.setState((prevState) => {
      return {
        users: prevState.users.filter((user) => user.id !== id),
      };
    });
  }

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map((user) => {
            return (
              <User
                key={user.id}
                name={user.name}
                id={user.id}
                onDeleteClick={this.deleteUser}
              />
            );
          })}
        </ul>
      </div>
    );
  }
}

export default App;
~~~

User.tsx
~~~
class User extends React.PureComponent<{
  name: string;
  id: number;
  onDeleteClick: (id: number) => void;
}> {
  constructor(props: {
    name: string;
    id: number;
    onDeleteClick: (id: number) => void;
  }) {
    super(props);
    this.onDeleteButtonClick = this.onDeleteButtonClick.bind(this);
  }

  onDeleteButtonClick() {
    this.props.onDeleteClick(this.props.id);
  }

  render() {
    const { name } = this.props;
    console.log(`${name} just rendered`);
    return (
      <li>
        <input
          type="button"
          value="Delete"
          onClick={this.onDeleteButtonClick}
        />
        {name}
      </li>
    );
  }
}

export default User;
~~~

### Passing Arguments to Event Handlers

Внутри цикла часто нужно передать дополнительный аргумент в обработчик события. Например, если `id` — это идентификатор строки, можно использовать следующие варианты:

~~~
<button onClick={(e) => this.deleteRow(id, e)}>Удалить строку</button>
<button onClick={this.deleteRow.bind(this, id)}>Удалить строку</button>
~~~

Две строки выше — эквивалентны.

Используя стрелочную функцию, необходимо передавать аргумент `e` явно, но с `bind` `e` передается автоматически.

### Функции, передаваемые обработчикам событий. Функциональные компоненты

Функции обработчика событий:

* Обычно определяются внутри ваших компонентов.
* Имеют имена, начинающиеся с `handle`, за которыми следует название события.

По соглашению принято называть обработчики событий как `handle`, за которым следует имя события. Вы часто будете видеть `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}` и так далее.

Функции, передаваемые обработчикам событий, должны передаваться, а не вызываться.

Когда `handleClick` передается как обработчик события `onClick`, это говорит React запомнить это и вызывать вашу функцию только тогда, когда пользователь нажимает кнопку.

`()` в конце `handleClick()` запускает функцию сразу во время рендеринга, без каких-либо кликов.

Убедитесь, что вы используете соответствующие теги HTML для обработчиков событий. Например, для обработки кликов используйте `<button onClick={handleClick}>` вместо `<div onClick={handleClick}>`. Использование `<button>` позволяет использовать встроенные функции браузера, такие как навигация с помощью клавиатуры.
