# ?Composition vs Inheritance

Некоторые компоненты не знают своих потомков заранее. Это особенно характерно для таких компонентов, как `Sidebar` или `Dialog`, которые представляют из себя как бы «коробку», в которую можно что-то положить.
Для таких компонентов мы рекомендуем использовать специальный проп `children`, который передаст дочерние элементы сразу на вывод:

~~~
function FancyBorder(props: {color: string, children: ReactNode[]}) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
~~~

Это позволит передать компоненту произвольные дочерние элементы, вложив их в JSX:

~~~
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Добро пожаловать
      </h1>
      <p className="Dialog-message">
        Спасибо, что посетили наш космический корабль!
      </p>
    </FancyBorder>
  );
}
~~~

Всё, что находится внутри JSX-тега `<FancyBorder>`, передаётся в компонент `FancyBorder` через проп `children`. Поскольку `FancyBorder` рендерит `{props.children}` внутри `<div>`, все переданные элементы отображаются в конечном выводе.

Иногда в компоненте необходимо иметь несколько мест для вставки. В таком случае можно придумать свой формат, а не использовать `children`:

~~~
function App() {
  return (
    <SplitPane
      left={
        <div>Contacts</div>
      }
      right={
        <div>Chat</div>
      } />
  );
}

function SplitPane(props: {left: ReactNode, right: ReactNode}) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}
~~~

В React нет никаких ограничений на то, что можно передать в качестве пропсов.

Композиция хорошо работает и для компонентов, определённых через классы:

~~~
class SignUpDialog extends React.Component<object, {login: string}> {
  constructor(props: object) {
    super(props);
    this.state = {login: ''};
  }

  handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Добро пожаловать на борт, ${this.state.login}!`);
  }

  render() {
    return (
      <Dialog title="Программа исследования Марса"
              message="Как к вам обращаться?">
        <input value={this.state.login}
              onChange={this.handleChange.bind(this)} />
        <button onClick={this.handleSignUp.bind(this)}>
          Запишите меня!
        </button>
      </Dialog>
    );
  }
}
~~~

~~~
function Dialog(props: {title: string, message: string, children: ReactNode[]}) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}
~~~

~~~
function FancyBorder(props: {color: string, children: ReactNode[]}) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
~~~
