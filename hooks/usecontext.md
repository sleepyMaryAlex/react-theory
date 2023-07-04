# ?useContext

Принимает объект контекста (значение, возвращённое из `React.createContext`) и возвращает текущее значение контекста для этого контекста.

Текущее значение контекста определяется пропом `value` ближайшего `<MyContext.Provider>` над вызывающим компонентом в дереве. Вызов `useContext(MyContext)` аналогичен выражению `static contextType = MyContext` в классе, либо компоненту `<MyContext.Consumer>`.

`useContext` возвращает значение контекста для вызывающего компонента. Он определяется как `value` переданный ближайшему `SomeContext.Provider` выше вызывающего компонента в дереве.

App.tsx
~~~
export const ThemeContext = React.createContext("red");

function App() {
  return (
    <ThemeContext.Provider value="green">
      <Photos />
    </ThemeContext.Provider>
  );
}
~~~

Photos.tsx
~~~
function Photos() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme }}>
      Я стилизован темой из контекста!
    </button>
  );
}
~~~

`useContext()` всегда ищет ближайшего `Provider` выше вызывающего его компонента. Он ищет вверх и не рассматривает провайдеров в компоненте, из которого вы вызываете `useContext()`.

~~~
const ThemeContext = createContext("red");

function App() {
  const theme = useContext(ThemeContext);
  return (
    <ThemeContext.Provider value="green">
      <button style={{ background: theme }}>
        Я стилизован темой из контекста! I'm red!
      </button>
    </ThemeContext.Provider>
  );
}
~~~

### Указание резервного значения по умолчанию

Если React не может найти провайдеров конкретного контекста в родительском дереве, значение контекста, возвращаемое `useContext()` будет равно значению по умолчанию, которое вы указали при создании этого контекста.

Значение по умолчанию никогда не меняется.

Часто вместо `null`, есть более значимое значение, которое вы можете использовать по умолчанию.

Таким образом, если вы случайно отрендерите какой-либо компонент без соответствующего провайдера, он не сломается.

### Переопределение контекста для части дерева

Вы можете переопределить контекст для части дерева, заключив эту часть в провайдер с другим значением.

App.tsx
~~~
export const ThemeContext = createContext("red");

function App() {
  return (
    <ThemeContext.Provider value="green">
      <Photos />
    </ThemeContext.Provider>
  );
}

export default App;
~~~

Photos.tsx
~~~
function Photos() {
  return (
    <ThemeContext.Provider value="yellow">
      <Button />
    </ThemeContext.Provider>
  );
}

export default Photos;
~~~

Button.tsx
~~~
function Button() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme }}>
      Я стилизован темой из контекста!
    </button>
  );
}

export default Button;
~~~

Вы можете вкладывать и переопределять провайдеры столько раз, сколько вам нужно.

### Контекст проходит через промежуточные компоненты

То, как работает контекст, может напомнить вам о наследовании свойств CSS. В CSS вы можете указать `color: blue` для `<div>`, и любой DOM-узел внутри него, независимо от того, насколько он глубок, унаследует этот цвет, если какой-либо другой DOM-узел в середине не переопределит его с помощью `color: green`. Точно так же в React единственный способ переопределить некоторый контекст, поступающий сверху, — это обернуть дочерние элементы в провайдер контекста с другим значением.

В CSS различные свойства, такие как цвет и фоновый цвет, не переопределяют друг друга. Вы можете установить красный цвет для всех `<div>`, не влияя на фоновый цвет. Точно так же разные контексты React не переопределяют друг друга. Каждый контекст, который вы создаете с помощью `createContext()`, полностью отделен от других и связывает вместе компоненты, использующие и предоставляющие этот конкретный контекст. Один компонент может без проблем использовать или предоставлять множество различных контекстов.
