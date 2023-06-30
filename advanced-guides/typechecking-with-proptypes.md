# ?Typechecking with PropTypes

По мере роста вашего приложения вы можете отловить много ошибок с помощью проверки типов. Для этого можно использовать расширения JavaScript вроде Flow и TypeScript. Но, даже если вы ими не пользуетесь, React предоставляет встроенные возможности для проверки типов. Для запуска этой проверки на пропсах компонента вам нужно использовать специальное свойство `propTypes`:

~~~
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Привет, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
~~~

В данном примере проверка типа показана на классовом компоненте, но она же может быть применена и к функциональным компонентам, или к компонентам, созданным с помощью `React.memo` или `React.forwardRef`.

Когда какой-то проп имеет некорректное значение, в консоли будет выведено предупреждение. По соображениям производительности `propTypes` проверяются только в режиме разработки.

Пример использования возможных валидаторов можно найти в документации.

С помощью `PropTypes.element` вы можете указать, что только один дочерний элемент может быть передан компоненту в качестве потомка:

~~~
<App>
  <p>Hi</p>
</App>

class App extends React.Component {
  render() {
    // Это должен быть ровно один элемент, иначе вы увидите предупреждение.
    const children = this.props.children;
    return <div>{children}</div>;
  }
}

App.propTypes = {
  children: PropTypes.element.isRequired,
};
~~~

Вы можете задать значения по умолчанию для ваших `props` с помощью специального свойства `defaultProps`:

~~~
class App extends React.Component {
  render() {
    return (
      <h1>Привет, {this.props.name}</h1>
    );
  }
}

App.defaultProps = {
  name: 'Незнакомец'
};
// Отрендерит "Привет, Незнакомец"
~~~

C ES2022 вы можете объявить `defaultProps` как статическое свойство внутри классового React компонента.

~~~
class App extends React.Component {
  static defaultProps = { name: "Незнакомец" };
  render() {
    return <h1>Привет, {this.props.name}</h1>;
  }
}
~~~

К функциональным компонентам можно также применять `PropTypes`.

~~~
import PropTypes from "prop-types";

function App({ name }) {
  return <div>Hello, {name}</div>;
}

App.propTypes = {
  name: PropTypes.string,
};

export default App;
~~~

Также к функциональным компонентам можно применить и `defaultProps`.

~~~
function App({ name }) {
  return <div>Hello, {name}</div>;
}

App.defaultProps = {
  name: "stranger",
};
~~~
