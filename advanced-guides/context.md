# ?Context

_Контекст_ предоставляет способ делиться  данными между компонентами без необходимости явно передавать пропсы через каждый уровень дерева.

Контекст разработан для передачи данных, которые можно назвать «глобальными» для всего дерева React-компонентов (например, текущий аутентифицированный пользователь, UI-тема или выбранный язык).

### `createContext(defaultValue)`

`defaultValue`: значение, которое вы хотите, чтобы контекст имел, когда в дереве над компонентом, который считывает контекст, нет соответствующего провайдера контекста. Если у вас нет значимого значения по умолчанию, укажите `null`.

`createContext` возвращает объект контекста.

Сам объект контекста не содержит никакой информации. Он представляет, какой контекст считывают или предоставляют другие компоненты. Как правило, вы будете использовать `SomeContext.Provider` в компонентах выше, чтобы указать значение контекста, и вызвать `useContext(SomeContext)` в компонентах ниже, чтобы прочитать его. Объект контекста имеет несколько свойств:

* `SomeContext.Provider` позволяет предоставить значение контекста компонентам.
* `SomeContext.Consumer` — это альтернативный и редко используемый способ чтения значения контекста.

### SomeContext.Provider

Оберните свои компоненты в провайдер контекста, чтобы указать значение этого контекста для всех компонентов внутри.

`value`: значение, которое вы хотите передать всем компонентам, читающим этот контекст внутри этого провайдера.

ThemeContext.tsx
~~~
export const ThemeContext = createContext('light');
~~~

App.tsx
~~~
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
~~~

~~~
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}
~~~

~~~
function ThemedButton() {
  const theme = useContext(ThemeContext);
  console.log(theme); // dark
  return <button className={theme} />;
}
~~~

Обычно контекст используется, если необходимо обеспечить доступ данных во многих компонентах на разных уровнях вложенности. По возможности не используйте его, так как это усложняет повторное использование компонентов.

Если вы хотите избавиться от передачи некоторых пропсов на множество уровней вниз, обычно композиция компонентов является более простым решением, чем контекст.

Один из способов решить эту проблему — передать вниз сам компонент `Avatar`, в случае чего промежуточным компонентам не нужно знать о пропсах `user` и `avatarSize`:

~~~
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}
~~~

~~~
// Теперь, это выглядит так:
<Page user={user} avatarSize={avatarSize}/>
// ... который рендерит ...
<PageLayout userLink={...} />
// ... который рендерит ...
<NavigationBar userLink={...} />
// ... который рендерит ...
{props.userLink}
~~~

Однако, иногда одни и те же данные должны быть доступны во многих компонентах на разных уровнях дерева и вложенности. Контекст позволяет распространить эти данные и их изменения на все компоненты ниже по дереву.

Управление текущим языком, UI темой или кешем данных — это пример тех случаев, когда реализация с помощью контекста будет проще использования альтернативных подходов.

### `Class.contextType`

В свойство класса `contextType` может быть назначен объект контекста, созданный с помощью `React.createContext()`. С помощью этого свойства вы можете использовать ближайшее и актуальное значение указанного контекста при помощи `this.context`. В этом случае вы получаете доступ к контексту, как во всех методах жизненного цикла, так и в рендер-методе.

ThemeContext.tsx
~~~
export const ThemeContext = React.createContext("dark");
~~~

App.tsx
~~~
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="light">
        <Comments />
      </ThemeContext.Provider>
    );
  }
}
~~~

Comments.tsx
~~~
class Comments extends React.Component {
  render() {
    return (
      <div>
        <Photos />
      </div>
    )
  }
}
~~~

Photos.tsx
~~~
class Photos extends React.Component {
  static contextType = ThemeContext;

  componentDidMount() {
    const theme = this.context;
    console.log(theme); //light
  }

  render() {
    return <div>Photos</div>;
  }
}
~~~

Вы можете подписаться только на один контекст, используя этот API.

### `Context.Consumer`

_Consumer_ — это React-компонент, который подписывается на изменения контекста. В свою очередь, использование этого компонента позволяет вам подписаться на контекст в функциональном компоненте.

ThemeContext.tsx
~~~
export const ThemeContext = React.createContext("dark");
~~~

App.tsx
~~~
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <ThemeContext.Consumer>
          {(theme) => <div>Our theme is: {theme}</div>}
        </ThemeContext.Consumer>
      </ThemeContext.Provider>
    );
  }
}
~~~

Эта функция принимает текущее значение контекста и возвращает React-компонент. Передаваемый аргумент `value` будет равен ближайшему (вверх по дереву) значению этого контекста, а именно пропу `value` компонента `Provider`. Если такого компонента `Provider` не существует, аргумент `value` будет равен значению `defaultValue`, которое было передано в `createContext()`.

### `Context.displayName`

Объекту `Context` можно задать строковое свойство `displayName`. React DevTools использует это свойство при отображении контекста.

~~~
const MyContext = React.createContext(/* некоторое значение */);
MyContext.displayName = 'MyDisplayName';
<MyContext.Provider> // "MyDisplayName.Provider" в DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" в DevTools
~~~

### Изменение контекста из вложенного компонента

Довольно часто необходимо изменить контекст из компонента, который находится где-то глубоко в дереве компонентов. В этом случае вы можете добавить в контекст функцию, которая позволит потребителям изменить значение этого контекста:

ThemeContext.tsx
~~~
export const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee",
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222",
  },
};

export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
~~~

App.tsx
~~~
class App extends React.Component<
  object,
  {
    theme: {
      foreground: string;
      background: string;
    };
    toggleTheme: () => void;
  }
> {
  toggleTheme: () => void;
  constructor(props: object) {
    super(props);
    this.toggleTheme = () => {
      this.setState((state) => ({
        theme: state.theme === themes.dark ? themes.light : themes.dark,
      }));
    };
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    };
  }

  render() {
    return (
      <ThemeContext.Provider value={this.state}>
        <Comments />
      </ThemeContext.Provider>
    );
  }
}
~~~

Comments.tsx
~~~
class Comments extends React.Component {
  render() {
    return (
      <ThemeContext.Consumer>
        {({ theme, toggleTheme }) => (
          <button
            onClick={toggleTheme}
            style={{
              backgroundColor: theme.background,
              color: theme.foreground,
            }}
          >
            Toggle Theme
          </button>
        )}
      </ThemeContext.Consumer>
    );
  }
}
~~~

Использование нескольких контекстов:

~~~
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
~~~

### Импорт и экспорт контекста из файла

Часто компонентам в разных файлах требуется доступ к одному и тому же контексту. Вот почему принято объявлять контексты в отдельном файле.

### Предостережения

Контекст использует сравнение по ссылкам, чтобы определить, когда запускать последующий рендер. Может происходить случайные повторные рендеры потребителей, при перерендере родителя Provider-компонента, потому что новый объект, передаваемый в `value`, будет создаваться каждый раз.

ThemeContext.tsx
~~~
export const ThemeContext = React.createContext({ something: "something" });
~~~

App.tsx
~~~
function App() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <>
      <button onClick={handleClick}>Click</button>
      <ThemeContext.Provider value={{ something: "something" }}>
        <ThemedButton />
      </ThemeContext.Provider>
    </>
  );
}
~~~

ThemedButton.tsx
~~~
const ThemedButton = memo(function ThemedButton() {
  const theme = useContext(ThemeContext);
  console.log("render");

  return <button>{theme.something}</button>;
});
~~~

Один из вариантов решения этой проблемы — хранение этого объекта в состоянии родительского компонента:

App.tsx
~~~
function App() {
  const [value] = useState({ something: "something" });
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <>
      <button onClick={handleClick}>Click</button>
      <ThemeContext.Provider value={value}>
        <ThemedButton />
      </ThemeContext.Provider>
    </>
  );
}
~~~

### What is the difference between the context API and prop drilling?

В React вы явно передаете данные из родительского компонента в дочерние через пропсы.

А что если дочерний компонент, которому нужны данные, оказывается глубоко вложенным? Это часто происходит, когда данные совместно используются многими различными компонентами — такие данные, как предпочтения локали, предпочтения темы или пользовательские данные (например, состояние аутентификации).

_Prop drilling_ - это ситуация, когда данные передаются от одного компонента через несколько взаимозависимых компонентов, пока вы не доберетесь до компонента, где данные необходимы.

И наоборот, Context API предоставляет нам центральное хранилище данных, к которому мы можем обращаться для получения данных из любого компонента без необходимости запрашивать его в качестве пропса.

### When shouldn't you use the context API?

Основным недостатком Context API является то, что каждый раз при изменении контекста все компоненты, потребляющие значение, перерисовываются. Это может иметь негативные последствия для производительности.

По этой причине вы должны использовать Context только для редко обновляемых данных, таких как настройки темы.

Итак, давайте поговорим о некоторых способах решения этой проблемы повторного рендеринга.

* Многие разработчики React говорят, что вам не стоит даже беспокоиться об оптимизации производительности, пока вы не увидите влияние на производительность.
* И Redux, и Mobx используют контекстный API, так чем же они помогают? Хранилище, совместно используемое этими библиотеками управления состоянием с контекстом, немного отличается от совместного использования состояния непосредственно с контекстом. Когда вы используете Redux и Mobx, работает алгоритм сравнения, который гарантирует повторную визуализацию только тех компонентов, которые действительно нуждаются в повторной визуализации.
* Используйте несколько контекстов (желательно разбивать на более мелкие) и сохраняйте состояние близко к его зависимым компонентам. Это означает, что если вам нужно поделиться некоторым состоянием, например формой, с несколькими компонентами, сделайте отдельный контекст только для формы и оберните компоненты формы в свой провайдер.
