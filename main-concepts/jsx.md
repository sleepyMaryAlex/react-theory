# ?JSX

_JSX_ (аббревиатура расшифровывается как _JavaScript XML_) — это расширение JavaScript, понятное только препроцессорам вроде _Babel_. После обнаружения препроцессором этот HTML-подобный текст преобразуется в обычные старые вызовы функций `React.createElement`:

~~~
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
~~~

То же самое:

~~~
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
~~~

JSX представляет собой объекты.

JSX допускает использование любых корректных JavaScript-выражений внутри фигурных скобок.

~~~
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;
~~~

JSX можно использовать внутри инструкций `if` и циклов `for`, присваивать переменным, передавать функции в качестве аргумента и возвращать из функции.

~~~
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
~~~

Чтобы использовать строковый литерал в качестве значения атрибута, используются кавычки:

~~~
const element = <a href="https://www.reactjs.org"> link </a>;
~~~

Если же в значении атрибута требуется указать JavaScript-выражение, то на помощь приходят фигурные скобки:

~~~
const element = <img src={user.avatarUrl}></img>;
~~~

Поскольку JSX ближе к JavaScript чем к HTML, React DOM использует стиль именования `camelCase` для свойств вместо обычных имён HTML-атрибутов.

Раньше вам приходилось импортировать React, потому что JSX конвертируется в обычный Javascript, который использует `React.createElement`. Но React представил новое преобразование JSX с выпуском React 17, которое автоматически преобразует JSX без использования. Это позволяет нам не импортировать React, однако вам нужно будет импортировать React, чтобы использовать хуки и другие экспорты, которые предоставляет React. Но если у вас есть простой компонент, вам больше не нужно импортировать React. Все преобразования JSX обрабатываются React без необходимости импортировать или добавлять что-либо.

Вы также можете ссылаться на React-компонент, используя запись через точку. Это удобно, если у вас есть модуль, который экспортирует много React-компонентов.

К примеру, если `MyComponents.DatePicker` является компонентом, то вы можете обратиться к нему напрямую, используя запись через точку:

App.tsx
~~~
function App() {
  return (
    <>
      <MyComponents.DatePicker color="blue" />
      <MyComponents.TimePicker color="red" />
    </>
  )
}
~~~

MyComponents.tsx
~~~
const MyComponents = {
  DatePicker: function DatePicker(props: { color: string }) {
    return <div>Представьте, что здесь цвет {props.color} виджета выбора даты.</div>;
  },
  TimePicker: function TimePicker(props: { color: string }) {
    return <div>Представьте, что здесь цвет {props.color} виджета выбора время.</div>;
  }
}
~~~

В качестве типа React-элемента нельзя использовать выражение.

~~~
const components = {
  component: MyComponent,
};

function App() {
  // Неправильно! JSX-тип не может являться выражением
  return <components['component'] />;
}
~~~

Чтобы исправить это, мы присвоим тип в переменную, начинающуюся с заглавной буквы:

~~~
const components = {
  component: MyComponent,
};

function App() {
  const Component = components['component'];
  return <Component />;
}
~~~

Вы можете передавать любые JavaScript-выражения как пропсы, обернув их в `{}`:

~~~
<MyComponent foo={1 + 2 + 3 + 4} />
~~~

Оператор `if` и цикл `for` не являются выражениями в JavaScript, поэтому их нельзя непосредственно использовать в JSX.

Вы можете передать строковый литерал как проп. Эти два выражения эквивалентны:

~~~
<MyComponent message="привет, мир" />
<MyComponent message={'привет, мир'} />
~~~

Если вы не передаёте значение в проп, то по умолчанию оно будет `true`. Эти два JSX выражения эквивалентны:

~~~
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
~~~

Если у вас уже есть пропсы внутри объекта `props` и вы хотите передать их в JSX, вы можете использовать оператор расширения `...`, чтобы передать весь объект с пропсами. Эти два компонента эквивалентны:

~~~
function App1() {
  return <Greeting firstName="Иван" lastName="Иванов" />;
}
~~~

~~~
function App2() {
  const props = {firstName: 'Иван', lastName: 'Иванов'};
  return <Greeting {...props} />;
}
~~~

Вы также можете выбрать конкретные пропсы, которые ваш компонент будет использовать, передавая все остальные пропсы с помощью оператора расширения.

~~~
const Button = props => {
  const { kind, ...other } = props; 
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("Кнопка нажата!")}>
        Привет, мир!
      </Button>
    </div>
  );
};
~~~

JSX удаляет пустые строки и пробелы в начале и конце строки. Новые строки, примыкающие к тегу будут удалены. Новые строки между строковых литералов сжимаются в один пробел. Следующие примеры кода рендерят одинаковый результат:

~~~
<div>Привет, мир</div>

<div>
  Привет, мир
</div>

<div>
  Привет,
  мир
</div>

<div>

  Привет, мир
</div>
~~~

Чтобы отобразить вложенные компоненты, можно указать несколько JSX-элементов в качестве дочерних.

~~~
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
~~~

Также React-компонент может возвращать массив элементов:

~~~
const MyComponent = () => {
  return [
    <p key="1">React can return multiple elements ❤️</p>,
    <p key="2">React can return multiple elements ❤️</p>,
    <p key="3">React can return multiple elements ❤️</p>,
  ];
};
~~~

или

~~~
class MyComponent extends React.Component {
  render() {
    return [
      <p key="1">React can return multiple elements ❤️</p>,
      <p key="2">React can return multiple elements ❤️</p>,
      <p key="3">React can return multiple elements ❤️</p>,
    ];
  }
}
~~~

Вы можете передать любое JavaScript-выражение как дочерний компонент, обернув его в `{}`. К примеру, эти выражения эквивалентны:

~~~
<MyComponent>Пример</MyComponent>
<MyComponent>{'Пример'}</MyComponent>
~~~

JavaScript-выражения могут быть использованы вместе с другими типами дочерних компонентов. Они могут рассматриваться как альтернатива шаблонным строкам:

~~~
function Hello(props) {
  return <div>Привет, {props.addressee}!</div>;
}
~~~

Обычно JavaScript-выражения, вставленные в JSX, будут приведены к строке, React-элементу или списку из всего этого. Тем не менее, `props.children` работает так же, как и любой другой проп, поэтому в него можно передавать любые типы данных, а не только те, которые React знает как рендерить. К примеру, если у вас есть пользовательский компонент, можно было бы передать колбэк в `props.children`:

App.tsx
~~~
function App() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>Это элемент списка с ключом {index}</div>}
    </Repeat>
  );
}
~~~

Repeat.tsx
~~~
function Repeat(props: {
  numTimes: number;
  children: (index: number) => ReactNode;
}) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}
~~~

Значения `false`, `null`, `undefined` и `true` — валидные дочерние компоненты. Просто они не рендерятся. Эти JSX-выражения будут рендерить одно и то же:

~~~
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
~~~

Этот подход может быть полезным для рендера по условию. Вот пример, где JSX рендерит `<Header />`, если `showHeader` равняется `true`:

~~~
<div>
  {showHeader && <Header />}
  <Content />
</div>
~~~

JSX строже, чем HTML. Вы должны закрыть такие теги, как `<br />`. Ваш компонент также не может возвращать несколько тегов JSX. Вы должны обернуть их в общий родитель, например `<div>...</div>` или пустую оболочку `<>...</>`.

### Is it possible to use React without JSX?

React можно использовать и без JSX, но большинство людей ценит его за наглядность при работе с UI.

JSX — это синтаксический сахар для функции `React.createElement(component, props, ...children)`, что означает, что вы можете напрямую вызывать `React.createElement`.

Например, вот код с JSX:

~~~
class App extends React.Component {
  render() {
    return (
      <div>
        <p>Hi</p>
        <ul>
          <li>Man</li>
          <li>Woman</li>
        </ul>
      </div>
    )
  }
}
~~~

Он может быть превращён в код без JSX:

~~~
class App extends React.Component {
  render() {
    return React.createElement(
      "div",
      null,
      React.createElement("p", null, "Hi"),
      React.createElement(
        "ul",
        null,
        React.createElement("li", null, "Man"),
        React.createElement("li", null, "Woman")
      )
    );
  }
}
~~~

Если вас интересуют другие примеры того, как JSX превращается в JavaScript, вы можете попробовать [онлайн-компилятор Babel](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA&debug=false&forceAllTransforms=false&modules=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=&version=7.22.5&externalPlugins=&assumptions=%7B%7D).

Кроме того, вы можете посмотреть на такие проекты как `react-hyperscript` и `hyperscript-helpers`, которые предлагают короткий синтаксис.

##### `react-hyperscript`

App.tsx
~~~
import AnotherComponent from "./components/AnotherComponent";
import createClass from "create-react-class";
import h from "react-hyperscript";

export default createClass({
  render() {
    return h("div.example", [
      h("h1#heading", "This is hyperscript"),
      h("h2", "creating React.js markup"),
      h(AnotherComponent, { children: [] }, [
        h("li", [h("a", { href: "http://whatever.com" }, "One list item")]),
        h("li", "Another list item"),
      ]),
    ]);
  },
});
~~~

AnotherComponent.tsx
~~~
class AnotherComponent extends React.Component<{children: ReactNode}> {
  render() {
    return <div>{this.props.children}</div>;
  }
}
~~~

##### `hyperscript-helpers`

~~~
// instead of writing
h('div')
// write
div()

// instead of writing
h('section#main', mainContents)
// write
section('#main', mainContents)
~~~

App.tsx
~~~
import AnotherComponent from "./components/AnotherComponent";
import createClass from "create-react-class";
import h from "react-hyperscript";
import hh from 'hyperscript-helpers';
const { div, h1, h2, li, a } = hh(h);

export default createClass({
  render() {
    return div([
      h1("#heading", "This is hyperscript"),
      h2("creating React.js markup"),
      h(AnotherComponent, { children: [] }, [
        li([a({ href: "http://whatever.com" }, "One list item")]),
        li("Another list item"),
      ]),
    ]);
  },
});
~~~

AnotherComponent.tsx
~~~
class AnotherComponent extends React.Component<{children: ReactNode}> {
  render() {
    return <div>{this.props.children}</div>;
  }
}
~~~
