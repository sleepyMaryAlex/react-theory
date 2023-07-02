# ?React optimizations

### `PureComponent`

Функциональные компоненты предотвращают создание экземпляров класса, уменьшая при этом общий размер пакета, так как он минимизируется лучше, чем классы.

С другой стороны, чтобы оптимизировать обновления пользовательского интерфейса, мы можем рассмотреть возможность преобразования функциональных компонентов в `PureComponent` класс. 

`React.PureComponent` делает поверхностное сравнение при изменении состояния. Т.е. если примитивные значения и их типы равны друг другу, `PureComponent` не запустит повторный рендер, а `Component` запустит.

Что касается объектов, то `PureComponent` сравнит ссылки. Если они равны, то не будет запускать рендер. В таком же случае `Component` запустит рендер.

Также `PureComponent` в отличие от `Component` не будет рендерить компонент если его пропсы не изменились. Например, состояние родительского поменялось, но мы не передавали пропсы в дочерний компонент, значит его пропсы не изменились, остались пустым объектом.

В большинстве случаев вы можете использовать `React.PureComponent` вместо написания собственного `shouldComponentUpdate`. Но он делает только поверхностное сравнение, поэтому его нельзя использовать, если пропсы и состояние могут измениться таким образом, который не сможет быть обнаружен при поверхностном сравнении.

Этот код работает неправильно:

App.tsx
~~~
class App extends React.Component<object, { words: string[] }> {
  constructor(props: object) {
    super(props);
    this.state = {
      words: ["word"],
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // Данная секция содержит плохой код и приводит к багам
    const words = this.state.words;
    words.push("word");
    this.setState({ words });


    // Правильный вариант
    this.setState({ words: [...this.state.words, "word"] });
  }

  render() {
    return (
      <div>
        <button type="button" onClick={this.handleClick}>
          Click
        </button>
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
~~~

ListOfWords.tsx
~~~
class ListOfWords extends React.PureComponent<{ words: string[] }> {
  render() {
    return <div>{this.props.words.join(", ")}</div>;
  }
}
~~~

Проблема в том, что `PureComponent` сделает сравнение по ссылке между старыми и новыми значениями `this.props.words`. Поскольку этот код мутирует массив `words` в методе `handleClick` компонента `WordAdder`, старые и новые значения `this.props.words` при сравнении по ссылке будут равны, даже если слова в массиве изменились. Даже если в массиве произойдут изменения, React не будет повторно отображать UI, поскольку это та же ссылка.

`ListOfWords` не будет обновляться, даже если он содержит новые слова, которые должны быть отрендерены.

### `shouldComponentUpdate`

Иногда части DOM перерисовываются, даже если они не изменились. В этом случае они являются побочным эффектом других частей, которые действительно меняются. Поэтому вы можете использовать метод жизненного цикла `shouldComponentUpdate`, который вызывается перед началом процесса ререндеринга.

Реализация этой функции по умолчанию возвращает `true`, указывая React выполнить обновление:

~~~
shouldComponentUpdate(nextProps, nextState) {
  if (this.props.color !== nextProps.color) {
    return true;
  }
  if (this.state.count !== nextState.count) {
    return true;
  }
  return false;
}
~~~

В этом коде `shouldComponentUpdate` — это простая проверка на наличие каких-либо изменений в `props.color` или `state.count`. Если эти значения не изменяются, то компонент не обновляется.

### Избегайте мутации объектов

Избегайте мутации объектов, которые вы используете как свойства или состояние, для повышения производительности.

Одной из главных особенностей React является его способность быстро определять, какие фрагменты данных изменились, и обновлять только затронутые компоненты пользовательского интерфейса.

Чтобы позволить React быстро распознавать новые изменения в вашем объекте состояния, вам следует избегать мутаций объекта и вместо этого создавать новый объект.

### `React.memo`

Если получается так, что нужно избавиться от лишнего рендера, когда мы обновили состояние новым, но таким же по сути массивом (т.е. эти массивы не ведут на одну и ту же ссылку, но одинаковы), то можно использовать `React.memo`, чтобы изменить условие рендеринга, например:

App.tsx
~~~
function App() {
  const [words, setWords] = useState<string[]>(["word"]);

  function handleClick() {
    setWords([...words]); // здесь мы создаем новый массив
  }

  return (
    <div>
      <button type="button" onClick={handleClick}>
        Click
      </button>
      <ListOfWords words={words} />
    </div>
  );
}
~~~

ListOfWords.tsx
~~~
function ListOfWords(props: { words: string[] }) {
  console.log("render");
  return <div>{props.words.join(", ")}</div>;
}

function areEqual(prevProps: { words: string[] }, nextProps: { words: string[] }) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
  return prevProps.toString() === nextProps.toString();
}

export default React.memo(ListOfWords, areEqual);
~~~

### Другие оптимизации

* Используйте `React.Fragments`, чтобы избежать дополнительных оболочек HTML-элементов.
* Избегайте определения встроенной функции в функции рендеринга.
* Избегайте использования индекса в качестве ключа.
* CSS-анимация вместо JS-анимации.
* Использование продакшен-сборки (можно проверить с помощью расширения `React Developer Tools` for Chrome).

> `npm build` создаст продакшен-сборку вашего приложения в папке `build/` вашего проекта. Помните, что это необходимо только перед деплоем на продакшен. Для обычной разработки используйте `npm start`.
