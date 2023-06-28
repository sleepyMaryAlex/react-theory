# ?Render props

Термин _«рендер-проп»_ относится к возможности компонентов React разделять код между собой с помощью пропа, значение которого является функцией.

Использование рендер-пропа может быть полезно для сквозных задач.

Компоненты — это основа повторного использования кода в React. Однако бывает неочевидно, как сделать, чтобы одни компоненты разделяли своё инкапсулированное состояние или поведение с другими компонентами, заинтересованными в таком же состоянии или поведении.

Например, следующий компонент отслеживает положение мыши в приложении:

~~~
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Перемещайте курсор мыши!</h1>
        <MouseWithCat />
      </div>
    );
  }
}
~~~

~~~
class MouseWithCat extends React.Component<object, { x: number; y: number }> {
  constructor(props: object) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event: React.MouseEvent<HTMLDivElement>) {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  }

  render() {
    return (
      <div style={{ height: "100vh" }} onMouseMove={this.handleMouseMove}>
        <Cat mouse={this.state} />
      </div>
    );
  }
}
~~~

~~~
class Cat extends React.Component<{ mouse: { x: number; y: number } }> {
  render() {
    const { mouse } = this.props;
    return (
      <img
        alt="cat"
        src={catImage}
        style={{
          position: "absolute",
          left: mouse.x,
          top: mouse.y,
          width: "50px",
        }}
      />
    );
  }
}
~~~

Допустим, нам нужно инкапсулировать поведение с возможностью повторного использования.

В примере выше, каждый раз когда мы хотим получить позицию мыши для разных случаев, нам требуется создавать новый компонент (т. е. другой экземпляр `<MouseWithCat>`), который рендерит что-то специально для этого случая.

Вот здесь рендер-проп нам и понадобится: вместо явного указания `<Cat>` внутри `<Mouse>` компонента, и трудозатратных изменений на выводе рендера, мы предоставляем `<Mouse>` функцию в качестве пропа, с которой мы используем динамическое определение того, что нужно передавать в рендер-проп.

~~~
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Перемещайте курсор мыши!</h1>
        <Mouse
          render={(mouse: { x: number; y: number }) => <Cat mouse={mouse} />}
        />
      </div>
    );
  }
}
~~~

~~~
class Mouse extends React.Component<
  { render: (mouse: { x: number; y: number }) => ReactNode },
  { x: number; y: number }
> {
  constructor(props: {
    render: (mouse: { x: number; y: number }) => ReactNode;
  }) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event: React.MouseEvent<HTMLDivElement>) {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  }

  render() {
    return (
      <div style={{ height: "100vh" }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}
~~~

~~~
class Cat extends React.Component<{ mouse: { x: number; y: number } }> {
  render() {
    const { mouse } = this.props;
    return (
      <img
        alt="cat"
        src={catImage}
        style={{
          position: "absolute",
          left: mouse.x,
          top: mouse.y,
          width: "50px",
        }}
      />
    );
  }
}
~~~

Компонент с рендер-пропом берёт функцию, которая возвращает React-элемент, и вызывает её вместо реализации собственного рендера. Иными словами, рендер-проп — функция, которая сообщает компоненту что необходимо рендерить.

Важно запомнить, что из названия паттерна _«рендер-проп»_ вовсе не следует, что для его использования вы должны обязательно называть проп `render`. На самом деле, любой проп, который используется компонентом и является функцией рендеринга, технически является и «рендер-пропом».

Несмотря на то, что в вышеприведённых примерах мы используем `render`, мы можем также легко использовать проп `children`!

~~~
<Mouse>
  {(mouse: { x: number; y: number }) => <Cat mouse={mouse} />}
</Mouse>
~~~

Один интересный момент касательно рендер-пропсов заключается в том, что вы можете реализовать большинство компонентов высшего порядка (HOC), используя обычный компонент вместе с рендер-пропом.

Например, если для вас предпочтительней HOC `withMouse` вместо компонента `<Mouse>`, вы можете создать обычный компонент `<Mouse>` вместе с рендер-пропом:

~~~
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Перемещайте курсор мыши!</h1>
        <Cat />
      </div>
    );
  }
}
~~~

~~~
class Cat extends React.Component<{ mouse: { x: number; y: number } }> {
  render() {
    const { mouse } = this.props;
    return (
      <img
        alt="cat"
        src={catImage}
        style={{
          position: "absolute",
          left: mouse.x,
          top: mouse.y,
          width: "50px",
        }}
      />
    );
  }
}

export default withMouse(Cat);
~~~

~~~
function withMouse(
  Component: ComponentType<{ mouse: { x: number; y: number } }>
) {
  return class extends React.Component {
    render() {
      return (
        <Mouse
          render={(mouse) => <Component {...this.props} mouse={mouse} />}
        />
      );
    }
  };
}
~~~

~~~
class Mouse extends React.Component<
  { render: (mouse: { x: number; y: number }) => ReactNode },
  { x: number; y: number }
> {
  constructor(props: {
    render: (mouse: { x: number; y: number }) => ReactNode;
  }) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event: React.MouseEvent<HTMLDivElement>) {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  }

  render() {
    return (
      <div style={{ height: "100vh" }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}
~~~

### Предостережения

Использование рендер-пропа может свести на нет преимущество, которое даёт `React.PureComponent`, если вы создаёте функцию внутри метода `render` (это касается не только рендер-пропа, но и любой стрелочной функции, когда мы передаем ее в компонент).

~~~
class App extends React.Component<object, { count: number }> {
  constructor(props: object) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      count: 1,
    };
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <>
        <Color render={(color: string) => <Paragraph color={color} />} />
        <p>{this.state.count}</p>
        <button type="button" onClick={this.handleClick}>
          increase count
        </button>
      </>
    );
  }
}
~~~

~~~
class Color extends React.PureComponent<
  { render: (color: string) => ReactNode },
  { color: string }
> {
  constructor(props: { render: (color: string) => ReactNode }) {
    super(props);
    this.setRed = this.setRed.bind(this);
    this.setBlue = this.setBlue.bind(this);
    this.state = { color: "red" };
  }

  setRed() {
    this.setState({
      color: "red",
    });
  }

  setBlue() {
    this.setState({
      color: "blue",
    });
  }

  render() {
    console.log("render");
    return (
      <div style={{ backgroundColor: this.state.color }}>
        <button type="button" onClick={this.setRed}>
          red
        </button>
        <button type="button" onClick={this.setBlue}>
          blue
        </button>
        {this.props.render(this.state.color)}
      </div>
    );
  }
}
~~~

~~~
class Paragraph extends React.Component<{ color: string }> {
  render() {
    return <p>{this.props.color}</p>;
  }
}
~~~

Чтобы решить эту проблему, вы можете определить проп как метод экземпляра, например так:

~~~
class App extends React.Component<object, { count: number }> {
  constructor(props: object) {
    super(props);
    this.renderTheCat = this.renderTheCat.bind(this);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      count: 1,
    };
  }

  renderTheCat(color: string) {
    return <Paragraph color={color} />;
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <>
        <Color render={this.renderTheCat} />
        <p>{this.state.count}</p>
        <button type="button" onClick={this.handleClick}>
          increase count
        </button>
      </>
    );
  }
}
~~~

Остальные компоненты остаются неизменными.
