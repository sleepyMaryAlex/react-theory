# ?useState

Компонентам часто необходимо изменить то, что отображается на экране в результате взаимодействия. Ввод в форму должен обновить поле ввода, нажатие «Далее» на карусели изображений должно изменить отображаемое изображение, нажатие «купить» должно поместить продукт в корзину.

Компоненты должны «запоминать» вещи: текущее входное значение, текущее изображение, корзину. В React такой тип памяти для конкретного компонента называется _состоянием_.

### Когда обычной переменной недостаточно

Вот компонент, который рендерит изображение скульптуры. Нажатие кнопки «Далее» должно отобразить следующую скульптуру, изменив индекс на `1`, затем на `2` и так далее. Однако это не сработает:

Gallery.tsx
~~~
import { sculptureList } from "./data";

function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}

export default Gallery;
~~~

data.ts
~~~
export const sculptureList = [
  {
    name: "Homenaje a la Neurocirugía",
    artist: "Marta Colvin Andrade",
    description:
      "Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.",
    url: "https://i.imgur.com/Mx7dA2Y.jpg",
    alt: "A bronze statue of two crossed hands delicately holding a human brain in their fingertips.",
  },
  {
    name: "Floralis Genérica",
    artist: "Eduardo Catalano",
    description:
      "This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.",
    url: "https://i.imgur.com/ZF6s192m.jpg",
    alt: "A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.",
  },
];
~~~

Обработчик события `handleClick` обновляет локальную переменную `index`. Но две вещи препятствуют тому, чтобы это изменение было видимым:

1. Локальные переменные не сохраняются между рендерами. Когда React визуализирует этот компонент во второй раз, он визуализирует его с нуля — он не учитывает никаких изменений в локальных переменных.
2. Изменения локальных переменных не вызывают рендеринг. React не понимает, что ему нужно снова отобразить компонент с новыми данными.

Хук `useState` обеспечивает эти две вещи.

### Анатомия `useState`

Мы вызываем `useState`, чтобы наделить наш функциональный компонент внутренним состоянием. Мы можем «сохранить» это внутреннее состояние между вызовами функции. Обычно переменные «исчезают» при выходе из функции. К переменным состояния это не относится, потому что их сохраняет React.

Вызов `useState` возвращает массив с двумя элементами, который содержит: текущее значение состояния и функцию для его обновления. Синтаксис деструктуризации массивов позволяет нам по-разному называть переменные состояния, которые мы объявляем при вызове `useState`.

> Есть соглашение, чтобы назвать эту пару как `const [something, setSomething]`. Вы можете назвать это как угодно, но соглашения облегчают понимание разных проектов.

Единственный аргумент `useState` — это начальное состояние.

~~~
const [age, setAge] = useState(42);
~~~

`useState` — это новый способ использовать те же возможности, что даёт `this.state` в классах. Заметьте, что в отличие от `this.state`, в нашем случае состояние может, но не обязано, быть объектом. Еще одно отличие от `this.setState` в классах, что обновление переменной состояния всегда замещает её значение, а не осуществляет слияние.

Хук состояния можно использовать в компоненте более одного раза и они будут независимы.

Cуществует один важный нюанс, связанный с его обновлением состояния с помощью `useState()`.

~~~
function DoubleIncreaser() {
  const [count, setCount] = useState(0);

  const doubleIncreaseHandler = () => {
    setCount(count + 1);
    setCount(count + 1);
  };

  return (
    <>
      <button onClick={doubleIncreaseHandler}>Double Increase</button>
      <div>Count: {count}</div>
    </>
  );
}

export default DoubleIncreaser;
~~~

Значение `count` будет увеличиваться на `1` после каждого клика. Когда `setCount(count + 1)` обновляет состояние, значение `count` не изменяется сразу. Вместо этого, React планирует обновление состояния и при следующем рендеринге в выражении `const [count, setCount] = useState(0)` хук присваивает `count` новое значение. Таким образом, обновление состояния с помощью `setValue(newValue)` в выражении `[value, setValue] = useState()` осуществляется асинхронно.

Однако, функция обновления состояния может принимать коллбэк в качестве аргумента для вычисления нового состояния на основе текущего.

~~~
const doubleIncreaseHandler = () => {
  setCount((actualCount) => actualCount + 1);
  setCount((actualCount) => actualCount + 1);
};
~~~

Значение `count` увеличится до `2`, как и ожидается.

Асинхронное обновление состояния характерно и для классовых компонентов.

~~~
class DoubleIncreaser extends React.Component<object, { count: number }> {
  constructor(props: object) {
    super(props);
    this.state = { count: 0 };
  }

  doubleIncrease = () => {
    // Работает!
    this.setState(({ count }) => ({
      count: count + 1,
    }));
    this.setState(({ count }) => ({
      count: count + 1,
    }));

    // Не работает!
    // this.setState({ count: this.state.count + 1 });
    // this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <>
        <button onClick={this.doubleIncrease}>Double Increase</button>
        <div>Count: {this.state.count}</div>
      </>
    );
  }
}
~~~

Состояние иммутабельно (неизменяемо) и доступно только для чтения. Пример - переменная `users`. Только хук `useState()`, а точнее функция `setUsers`, может присвоить ей новое значение.

~~~
const [users, setUsers] = React.useState<string[]>([]);
useEffect(() => {
  const startFetching = async () => {
    const response = await fetch("/users");
    const fetchedUsers = await response.json();
    users = fetchedUsers; // Неправильно! users доступна только для чтения
    setUsers(fetchedUsers); // Правильно
    console.log(users); // => [], обновляется не сразу
    console.log(fetchedUsers); // => ['John', 'Jane', 'Alice', 'Bob']
  };
  startFetching();
}, []);
~~~

Нельзя изменять свойства состояния напрямую, в этом случае повторного рендера компонента происходить не будет.

Помимо этого, `setState` — асинхронный метод, который создает очередь изменений состояния компонента. Меняя состояние напрямую, минуя очередь, мы ломаем логику обработки данных (сравнение предыдущего и следующего состояния), что может сказаться на правильной работе методов жизненного цикла React-компонента.

### Предоставление компоненту нескольких переменных состояния

В одном компоненте вы можете иметь столько переменных состояния любого типа, сколько захотите.

Рекомендуется иметь несколько переменных состояния, если их состояние не связано. Но если вы обнаружите, что часто меняете две переменные состояния вместе, может быть проще объединить их в одну.

### Состояние изолированное

Состояние является локальным для экземпляра компонента. Другими словами, если вы рендерите один и тот же компонент дважды, каждая копия будет иметь полностью изолированное состояние! Изменение одного из них не повлияет на другой.
