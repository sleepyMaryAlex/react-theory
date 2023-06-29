# ?React without ES6

Обычно компонент React определяется как простой JavaScript-класс. Если приложение без ES6, то можете использовать модуль `create-react-class`. API ES6-классов похож на `createReactClass()` за некоторыми исключениями.

С помощью функций и классов ES6 `defaultProps` определяется как свойство самого компонента и вы можете определять начальное состояние через `this.state` в конструкторе. Также вам придётся явно использовать `.bind(this)` в конструкторе (без привязки `handleClick` вызовется, но `this` будет `undefined`):

~~~
class App extends React.Component<{ name: string }, { age: number }> {
  constructor(props: { name: string }) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      age: 20,
    };
  }

  static defaultProps = {
    name: "Vasilisa",
  };

  handleClick() {
    console.log(this.props.name);
  }

  render() {
    return (
      <>
        <button onClick={this.handleClick}>Say HI</button>
        <p>
          Name: {this.props.name}. Age {this.state.age}.
        </p>
      </>
    );
  }
}
~~~

При использовании `createReactClass()` вам нужно определить метод `getDefaultProps()` в переданном объекте и вам придётся отдельно реализовать метод `getInitialState`, который возвращает начальное состояние. И не нужно использовать `.bind(this)`:

~~~
const createReactClass = require("create-react-class");

const App = createReactClass({
  getDefaultProps() {
    return {
      name: "Vasilisa",
    };
  },

  getInitialState() {
    return { age: 20 };
  },

  handleClick() {
    console.log(this.props.name);
  },

  render() {
    return (
      <>
        <button onClick={this.handleClick}>Say HI</button>
        <p>
          Name: {this.props.name}. Age {this.state.age}.
        </p>
      </>
    );
  },
});
~~~
