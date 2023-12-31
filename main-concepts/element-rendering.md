# ?Element rendering

Элементы — мельчайшие кирпичики React-приложений.

В отличие от DOM-элементов, элементы React — это простые объекты, не отнимающие много ресурсов. React DOM обновляет DOM, чтобы он соответствовал переданным React-элементам.

#### Element rendering in DOM

Допустим, в вашем HTML-файле есть `<div>`. Мы назовём его «корневым» узлом DOM, так как React DOM будет управлять его содержимым. Обычно в приложениях, написанных полностью на React, есть только один корневой элемент.

При встраивании React в существующее приложение вы можете рендерить во столько независимых корневых элементов, во сколько посчитаете нужным. Для рендеринга React-элемента, сперва передайте DOM-элемент в `ReactDOM.createRoot()`, далее передайте с React-элементом в `root.render()`:

~~~
const root = ReactDOM.createRoot(
  document.getElementById('root')
);
const element = <h1>Hello, world</h1>;
root.render(element);
~~~

#### Update elements on a page

Элементы React иммутабельны. После создания элемента нельзя изменить его потомков или атрибуты. Элемент похож на кадр в фильме: он отражает состояние интерфейса в конкретный момент времени.

React DOM сравнивает элемент и его дочернее дерево с предыдущей версией и вносит в DOM только минимально необходимые изменения.

### Conditional rendering

React позволяет разделить логику на независимые компоненты. Эти компоненты можно показывать или прятать в зависимости от текущего состояния.

Условный рендеринг в React работает так же, как условные выражения работают в JavaScript. Бывает нужно объяснить React, как состояние влияет на то, какие компоненты требуется скрыть, а какие — отрендерить, и как именно. В таких ситуациях используйте тернарный оператор JavaScript или выражения с `if`.

Элементы React можно сохранять в переменных. Это может быть удобно, когда какое-то условие определяет, надо ли рендерить одну часть компонента или нет, а другая часть компонента остаётся неизменной.

~~~
class LoginControl extends React.Component<object, {isLoggedIn: boolean}> {
  constructor(props: object) {
    super(props);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    let button;
    if (this.state.isLoggedIn) {
      button = <button onClick={this.handleLogoutClick.bind(this)}>Logout</button>;
    } else {
      button = <button onClick={this.handleLoginClick.bind(this)}>Login</button>;
    }
    return (
      <div>
        {button}
      </div>
    );
  }
}
~~~

Также, можно использовать логический оператор `&&`, которым можно удобно вставить элемент в зависимости от условия:

~~~
function Mailbox(props) {
  return (
    <div>
      <h1>Здравствуйте!</h1>
      {props.unreadMessages.length > 0 &&
        <h2>
          У вас {props.unreadMessages.length} непрочитанных сообщений.
        </h2>
      }
    </div>
  );
}
~~~

Приведённый выше вариант работает корректно, потому что в JavaScript-выражение `true && expression` всегда вычисляется как `expression`, а выражение `false && expression` — как `false`.

То есть, если условие истинно (`true`), то элемент, идущий непосредственно за `&&`, будет передан на вывод. Если же оно ложно (`false`), то React проигнорирует и пропустит его.

В редких случаях может потребоваться позволить компоненту спрятать себя, хотя он уже и отрендерен другим компонентом. Чтобы этого добиться, верните `null` вместо того, что обычно возвращается на рендеринг.

~~~
function WarningBanner(props: { warn: boolean }) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Предупреждение!
    </div>
  );
}

class Page extends React.Component<object, { showWarning: boolean }> {
  constructor(props: object) {
    super(props);
    this.state = {showWarning: true};
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick.bind(this)}>
          {this.state.showWarning ? 'Спрятать' : 'Показать'}
        </button>
      </div>
    );
  }
}
~~~

Сам факт возврата `null` из метода `render` компонента никак не влияет на срабатывание методов жизненного цикла компонента. Например, `componentDidUpdate` будет всё равно вызван.

### When is a component rendered?

После первичного рендеринга и отображения приложения в DOM дереве, повторный рендер может быть вызван по одной из следующих причин.

Для классовых компонентов:

* `this.setState()`
* `this.forceUpdate()`

Для функциональных компонентов:

* `useState`
* `useReducer`

Для всех:

* Рендер родителя вызовет рендер всех его дочерних элементов (стандартное поведение, но возможна оптимизация)
* Повторный рендеринг будет вызван, если повторно запустить `ReactDOM.render(<App />)`, что эквивалентно `forceUpdate()` на корневом компоненте.

### How not to render on props change?

Для предотвращения лишнего рендера могут быть использованы:

* `React.Component.shouldComponentUpdate` — метод жизненного цикла классового компонента, если он вернет `false` то рендер не будет запущен.
* `React.PureComponent` — класс, реализующий типовой `shouldComponentUpdate`.
* `React.memo()` — HOC, который предотвращает повторный рендер, если входные `props` не изменились.

Используйте `React.memo()` на родительском компоненте, чтобы остановить каскадный рендер всех дочерних компонентов.

### Is it OK to use arrow functions in render methods?

#### Как привязать функцию к экземпляру компонента

Привязка в конструкторе (ES2015):

~~~
class Foo extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log('По кнопке кликнули');
  }

  render() {
    return <button onClick={this.handleClick}>Нажми на меня</button>;
  }
}
~~~

Привязка в методе `render()`:

~~~
render() {
  return <button onClick={this.handleClick.bind(this)}>Нажми на меня</button>;
}
~~~

Свойства класса (ES2022):

~~~
class Foo extends Component {
  handleClick = () => {
    console.log('По кнопке кликнули');
  };

  render() {
    return <button onClick={this.handleClick}>Нажми на меня</button>;
  }
}
~~~

Но почему лучше не использовать стрелочные функции для всех свойств класса:

* Поскольку они являются свойствами, а не методами, они не существуют в прототипе.
* После транспиляции стрелочная функция в свойстве будет перемещена в конструктор.
* Не могут использоваться в качестве конструктора.

Стрелочная функция определяется только при инициализации конструктором, а не в прототипе. Таким образом,  cтрелочных функций в свойствах класса не будет в прототипе и мы не сможем вызывать их с помощью `super`.

Поскольку стрелочные функции являются свойствами, а не методами, они не существуют в прототипе. Это предотвращает повторное использование браузером одной и той же функции при рендеринге нескольких копий одного и того же элемента (каждая копия элемента будет запускать другую функцию рендеринга вместо одной и той же функции с разными `this` каждый раз), что затрудняет оптимизацию движком Javascript.

Даже там, где поддерживаются свойства стрелочных функций (TypeScript, babel с плагином свойств классов), рекомендуется писать все как методы, чтобы они существовали в прототипе. Если какой-либо из ваших методов необходимо указать в качестве аргумента или свойства элемента JSX, рекомендуется сделать это `this.methodName = this.methodName.bind(this)` в конструкторе, создав локальную версию общего метода-прототипа.

PS: все функции жизненного цикла уже привязаны, точнее они вызываются в правильном контексте.

Стрелочная функция в `render()`:

~~~
render() {
  return <button onClick={() => this.handleClick()}>Нажми на меня</button>;
}
~~~

Использование стрелочной функции в `render()` создаёт новую функцию при каждой отрисовке компонента, что может нарушать оптимизации, использующие строгое сравнение для определения идентичности.

### Почему мы пишем super(props)?

Почему мы вызываем `super`? Можем ли мы не вызывать его? Если нам нужно вызвать его, что произойдет, если мы не передадим `props`?

В JavaScript `super` относится к конструктору родительского класса. (В нашем примере это указывает на реализацию `React.Component`).

~~~
class Checkbox extends React.Component {
  constructor(props) {
    // Нельзя использовать `this` здесь
    super(props);
    // А вот здесь можно
    this.state = { isOn: true };
  }
  // ...
}
~~~

Есть веская причина, по которой JavaScript принудительно вызывает родительский конструктор до того, как вы используете `this` в нем. Рассмотрим иерархию классов:

~~~
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // Это запрещено
    super(name);
  }
  greetColleagues() {
    alert('Good morning folks!');
    alert('My name is ' + this.name + ', nice to meet you!');
  }
}
~~~

`this.greetColleagues()` вызывается до того, как вызов `super()` смог установить `this.name`. Так что `this.name` еще даже не определено!

Чтобы избежать таких ловушек, JavaScript требует, чтобы, если вы хотите использовать `this` в конструкторе, вы должны сначала вызвать `super`. Пусть родитель делает свое дело! И это ограничение распространяется и на компоненты React, определенные как классы:

~~~
constructor(props) {
  super(props);
  // Теперь можно использовать this здесь
  this.state = { isOn: true };
}
~~~

Но это оставляет нас с другим вопросом: зачем передавать `props`?

Вы можете подумать, что передача props в `super` необходима. Но каким-то образом, даже если вы вызовете `super()` без аргумента `props`, вы все равно сможете получить доступ к `this.props` в рендеринге и других методах.

Как это работает? Оказывается, React также присваивает `props` экземпляру сразу после вызова вашего конструктора:

~~~
// Внутри React
const instance = new YourComponent(props);
instance.props = props;
~~~

Таким образом, даже если вы забудете передать `props` в `super()`, React все равно установит их сразу после этого.

Значит ли это, что вы можете просто написать `super()` вместо `super(props)`?

Конечно, React позже назначит `this.props` после вызова вашего конструктора. Но `this.props` по-прежнему будет `undefined` между вызовом `super` и концом вашего конструктора:

~~~
class Button extends React.Component {
  constructor(props) {
    super(); // Мы забыли передать props
    console.log(this.props); // undefined 
  }
  // ...
}
~~~
