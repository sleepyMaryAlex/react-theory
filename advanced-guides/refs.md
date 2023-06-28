# ?Refs

_Рефы_ предоставляют способ доступа к узлам DOM или элементам React, созданным в методе рендеринга.

### What is the difference between refs and state variables?

И refs, и переменные состояния позволяют сохранять значения между рендерингами; однако только переменные состояния вызывают повторный рендеринг.

В то время как рефы традиционно использовались и до сих пор используются для прямого доступа к элементам DOM, становится все более распространенным использование рефов в функциональных компонентах для сохранения значений между рендерингами, которые не должны запускать повторный рендеринг при обновлении значения.

Поэтому не так много причин использовать refs в компонентах класса, потому что более естественно хранить эти значения в полях, принадлежащих экземпляру класса, и они будут сохраняться между рендерингами независимо.

### When is the best time to use refs?

Используйте рефы только в случае необходимости, чтобы избежать нарушения инкапсуляции.

Рефы в основном используются одним из двух способов:

1. Одним из способов использования рефов является прямой доступ к элементу DOM для управления им — например, управление фокусом, выделение текста или воспроизведение медиа, императивный вызов анимаций, интеграция со сторонними DOM-библиотеками.
2. Второе использование рефов — в функциональных компонентах, где они иногда являются хорошим выбором утилиты для сохранения значений между рендерингами без запуска повторного рендеринга компонента при изменении значения. В классовых компонентах лучше хранить эти значения в полях, принадлежащих экземпляру класса, и они будут сохраняться между рендерингами независимо.

~~~
// храним значение в полях класса
class App extends React.Component<object, { renderIndex: number }> {
  insteadOfCreateRef: number;
  constructor(props: object) {
    super(props);
    this.state = {
      renderIndex: 1,
    };
    this.insteadOfCreateRef = 1;
  }

  render() {
    console.log(this.insteadOfCreateRef);
    this.insteadOfCreateRef = this.state.renderIndex;
    return (
      <>
        Current render index: {this.state.renderIndex}
        <br />
        {this.insteadOfCreateRef}
        <button
          onClick={() =>
            this.setState({ renderIndex: this.state.renderIndex + 1 })
          }
        >
          Cause re-render
        </button>
      </>
    );
  }
}
~~~

~~~
// useRef для сохранения значения
function App() {
  const [renderIndex, setRenderIndex] = useState(1);
  const ref = useRef<number>(0);

  console.log(ref.current);
  ref.current = renderIndex;

  return (
    <>
      Current render index: {renderIndex}
      {ref.current}
      <button onClick={() => setRenderIndex((prev) => prev + 1)}>
        Cause re-render
      </button>
    </>
  );
}
~~~

`createRef` для сохранения значения не сработает, так как `createRef` возвращает объект `{ readonly current: T }`, соответственно вообще не предполагается изменение значения `current`.

### `useRef` vs `createRef`

Специальная функция `createRef`, она возвращает `{ current: null }` и по сути является синтаксическим сахаром. Когда атрибут `ref` используется на HTML-элементе, свойство `current` созданного рефа в конструкторе с помощью `React.createRef()` получает соответствующий DOM-элемент. React присвоит DOM-элемент свойству `current` при монтировании компонента и присвоит обратно значение `null` при размонтировании.

`useRef` - это хук, возвращает `MutableRefObject`, то есть объект вида `{ current: T }`, который можно изменять, а вот `createRef` возвращает объект `{ readonly current: T }`, соответственно не предполагается изменение значения current.

`useRef` сохраняет существующую ссылку между повторными рендерами. `createRef` - каждый раз создает новую ссылку. В компоненте на основе класса вы обычно помещаете ссылку в свойство экземпляра во время создания (например, `this.input = createRef()`). У вас нет этой опции в функциональном компоненте. `useRef` заботится о возврате той же ссылки каждый раз, что и при первоначальном рендеринге.

`useRef` используется в функциональных компонентах. `createRef` используется в компонентах класса, также может использоваться в функциональных компонентах, но может показывать несоответствия. 

### Не злоупотребляйте `ref`

Избегайте использования рефов в ситуациях, когда задачу можно решить декларативным способом.

~~~
// Плохо
function Item() {
  const ref = useRef<HTMLDivElement>(null);

  function open() {
    (ref.current as HTMLDivElement).style.display = "flex";
  }

  function close() {
    (ref.current as HTMLDivElement).style.display = "none";
  }

  return (
    <>
      <div ref={ref}>div</div>
      <button onClick={open}>open</button>
      <button onClick={close}>close</button>
    </>
  );
}
~~~

~~~
// Хорошо
function Item() {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <>
      <div style={{ display: isOpen ? "flex" : "none" }}>div</div>
      <button onClick={() => setIsOpen(true)}>open</button>
      <button onClick={() => setIsOpen(false)}>close</button>
    </>
  );
}
~~~

### Добавление рефа к классовому компоненту

Для того чтобы произвести имитацию клика по `CustomTextInput` сразу же после монтирования, можно использовать реф, чтобы получить доступ к пользовательскому `<input>` и явно вызвать его метод `focusTextInput`:

~~~
class App extends React.Component {
  textInput: React.RefObject<CustomTextInput>;
  constructor(props: object) {
    super(props);
    this.textInput = React.createRef<CustomTextInput>();
  }

  componentDidMount() {
    console.log(this.textInput.current);
    this.textInput.current?.focusTextInput();
  }

  render() {
    return <CustomTextInput ref={this.textInput} />;
  }
}
~~~

~~~
class CustomTextInput extends React.Component {
  textInput: React.RefObject<HTMLInputElement>;
  constructor(props: object) {
    super(props);
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    this.textInput.current?.focus();
  }

  render() {
    return (
      <div>
        <input placeholder="input" type="text" ref={this.textInput} />
        <input
          type="button"
          value="Фокус на текстовом поле"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
~~~

Обратите внимание, что это сработает только в том случае, если `CustomTextInput` объявлен как классовый компонент, насчет `App` - неважно. Когда атрибут `ref` используется на классовом компоненте, свойство `current` объекта-рефа получает экземпляр смонтированного компонента.

Если вам нужно передать реф на функциональный компонент, можете воспользоваться `forwardRef` (возможно вместе с `useImperativeHandle`), либо превратить его в классовый компонент.

### Колбэк-рефы

~~~
class App extends React.Component {
  nameInput: HTMLInputElement | null | undefined;

  componentDidMount() {
    this.nameInput?.focus();
  }

  render() {
    return (
      <input
        placeholder="Will focus"
        ref={(input) => {
          this.nameInput = input;
        }}
      />
    );
  }
}
~~~

React вызовет `ref` колбэк с DOM-элементом при монтировании компонента, а также вызовет его со значением `null` при размонтировании. Рефы будут хранить актуальное значение перед вызовом методов `componentDidMount` или `componentDidUpdate`.

Вы можете передавать колбэк-рефы между компонентами точно так же, как и объектные рефы, созданные через `React.createRef()`.

##### Предостережения насчёт колбэк-рефов.

Если `ref` колбэк определён как встроенная функция, колбэк будет вызван дважды во время обновлений: первый раз со значением `null`, а затем снова с DOM-элементом. Это связано с тем, что с каждым рендером создаётся новый экземпляр функции, поэтому React должен очистить старый реф и задать новый. Такого поведения можно избежать, если колбэк в `ref` будет определён с привязанным к классу контекстом, но, заметим, что это не будет играть роли в большинстве случаев.

Пример:

~~~
class App extends React.Component<object, { count: number }> {
  textInput: HTMLInputElement | null | undefined;
  constructor(props: object) {
    super(props);
    this.state = {
      count: 0,
    };
    this.handleClick = this.handleClick.bind(this);
  }

  componentDidMount() {
    this.textInput?.focus();
  }

  componentDidUpdate() {
    this.textInput?.focus();
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <>
        <input
          placeholder="Will focus"
          ref={(input: HTMLInputElement | null) => {
            this.textInput = input;
            console.log(input);
          }}
        />
        <p>{this.state.count}</p>
        <button onClick={this.handleClick}>Increase count</button>
      </>
    );
  }
}
~~~

Как можно избежать создания нового экземпляра функции:

~~~
class App extends React.Component<object, { count: number }> {
  textInput: HTMLInputElement | null | undefined;
  constructor(props: object) {
    super(props);
    this.state = {
      count: 0,
    };
    this.handleClick = this.handleClick.bind(this);
    this.setTextInputRef = this.setTextInputRef.bind(this);
  }

  componentDidMount() {
    this.textInput?.focus();
  }

  componentDidUpdate() {
    this.textInput?.focus();
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  setTextInputRef(input: HTMLInputElement | null) {
    this.textInput = input;
    console.log(input);
  }

  render() {
    return (
      <>
        <input placeholder="Will focus" ref={this.setTextInputRef} />
        <p>{this.state.count}</p>
        <button onClick={this.handleClick}>Increase count</button>
      </>
    );
  }
}
~~~

### `forwardRef`

`forwardRef` позволяет автоматически передавать `ref` компонента одному из его дочерних элементов.

В маленьких, повторно используемых компонентах, чтобы управлять фокусом, выделением и анимациями этих компонентов, придётся получить доступ к их DOM-узлам.

Перенаправление рефов позволяет взять `ref` из атрибутов компонента, и передать («перенаправить») его одному из дочерних компонентов.

Обычные функциональные или классовые компоненты не получают `ref` в качестве аргумента или пропа, будет ошибка. `ref` это не проп. Подобно `key`, React обрабатывает `ref` особым образом.

App.tsx
~~~
function App() {
  const firstTextareaRef = createRef<HTMLTextAreaElement>();
  const secondTextareaRef = createRef<HTMLTextAreaElement>();

  useEffect(() => {
    (firstTextareaRef.current as HTMLTextAreaElement).focus();
  }, []);

  return (
    <div>
      <TextArea ref={firstTextareaRef} text="First" />
      <TextArea ref={secondTextareaRef} text="Second" />
    </div>
  );
}
~~~

TextArea.tsx
~~~
const TextArea = React.forwardRef<HTMLTextAreaElement, { text: string }>(
  (props, ref) => {
    return (
      <textarea title="textarea" ref={ref}>
        {props.text}
      </textarea>
    );
  }
);
~~~

Может немного сбить с толку, потому что порядок общих параметров (`ref` и затем `props`) противоположен порядку параметров функции (`props` и затем `ref`).

Особенно полезным перенаправление может оказаться в компонентах высшего порядка (также известных как HOC).

Компонент высшего порядка `logProps` передаёт все пропсы в компонент, который он оборачивает. HOC будет выводить в консоль все пропсы, переданные в наш компонент с кнопкой. Если вы укажете реф для HOC, он привяжется к ближайшему корню контейнера, а не к переданному в HOC компоненту.

К счастью, мы можем явно перенаправить рефы на компонент `FancyButton` внутри HOC при помощи API `React.forwardRef`. В `React.forwardRef` передаётся функция рендеринга, которая принимает аргументы `props` и `ref`, а возвращает узел React. 

App.tsx
~~~
function App() {
  const ref = createRef<HTMLButtonElement>();

  return <FancyButton ref={ref} label="Click me" />;
}
~~~

logProps.tsx
~~~
function logProps(
  Component: ComponentType<{
    label: string;
    ref: React.ForwardedRef<HTMLButtonElement>;
  }>
) {
  class LogProps extends React.Component<{
    label: string;
    forwardedRef: React.ForwardedRef<HTMLButtonElement>;
  }> {
    componentDidUpdate() {
      console.log("new props:", this.props);
    }

    render() {
      const { forwardedRef, ...rest } = this.props;

      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  return React.forwardRef<HTMLButtonElement, { label: string }>(
    (props, ref) => {
      return <LogProps {...props} forwardedRef={ref} />;
    }
  );
}
~~~

FancyButton.tsx
~~~
const FancyButton = React.forwardRef<HTMLButtonElement, { label: string }>(
  (props, ref) => {
    return <button ref={ref}>{props.label}</button>;
  }
);

export default logProps(FancyButton);
~~~

В `React.forwardRef` передаётся функция рендеринга. Эта функция определяет, как будет называться компонент в инструментах разработки. Если присвоить имя функции рендеринга, то оно появится в названии компонента в инструментах разработки (например, `«ForwardRef(myFunction)»`):

~~~
const FancyButton = React.forwardRef<HTMLButtonElement, { label: string }>(
  function myFunction(props, ref) {
    return <button ref={ref}>{props.label}</button>;
  }
);
~~~

Можно передать несколько рефов дочернему без `forwardRef`:

~~~
function App() {
  const ref1 = useRef<HTMLDivElement>(null);
  const ref2 = useRef<HTMLDivElement>(null);

  return <Item allRefs={{ ref1, ref2 }} />;
}
~~~

~~~
function Item(props: {
  allRefs: {
    ref1: React.RefObject<HTMLDivElement>;
    ref2: React.RefObject<HTMLDivElement>;
  };
}) {
  function handleClick(index: number) {
    index === 0
      ? ((props.allRefs.ref1.current as HTMLDivElement).style.background =
          "red")
      : ((props.allRefs.ref2.current as HTMLDivElement).style.background =
          "red");
  }

  return (
    <>
      <div ref={props.allRefs.ref1} onClick={() => handleClick(0)}>
        div1
      </div>
      <div ref={props.allRefs.ref2} onClick={() => handleClick(1)}>
        div2
      </div>
    </>
  );
}
~~~

Или передать несколько рефов с помощью `useImperativeHandle`:

~~~
interface IRef {
  focusMyInput1: () => void;
  focusMyInput2: () => void;
}

function App() {
  const inputsRef = useRef<IRef>(null);

  return (
    <>
      <Item ref={inputsRef} />
      <button onClick={() => inputsRef.current?.focusMyInput1()}>Focus Input1</button>
      <button onClick={() => inputsRef.current?.focusMyInput2()}>Focus Input2</button>
    </>
  );
}
~~~

~~~
const Item = React.forwardRef((props, ref) => {
  const refInput1 = useRef<HTMLInputElement>(null);
  const refInput2 = useRef<HTMLInputElement>(null);

  useImperativeHandle(ref, () => ({
    focusMyInput1: () => {
      refInput1.current?.focus();
    },
    focusMyInput2: () => {
      refInput2.current?.focus();
    },
  }));

  return (
    <>
      <input placeholder="input" ref={refInput1} />
      <input placeholder="input" ref={refInput2} />
    </>
  );
});
~~~

В React данные передаются от родительских к дочерним компонентам через свойства, известные как однонаправленный поток данных. Родительский компонент не может напрямую вызвать функцию, определенную в дочернем компоненте, или получить значение.

В определенных обстоятельствах мы хотим, чтобы наш родительский компонент обращался к дочернему компоненту, получая данные, которые происходят в дочернем компоненте, для собственного использования. Мы можем добиться такого типа потока данных с помощью хука `useImperativeHandle`, который позволяет нам предоставлять значение, состояние или функцию внутри дочернего компонента родительскому компоненту через `ref`.

Вы также можете решить, к каким свойствам может получить доступ родительский компонент, тем самым сохраняя приватную область действия дочернего компонента.

`useImperativeHandle` следует использовать с `forwardRef`.
