# ?useEffect

Используя этот хук, вы говорите React сделать что-то после рендера.

С помощью хука эффекта `useEffect` вы можете выполнять побочные эффекты (загрузка данных, оформление подписки и изменение DOM вручную) из функционального компонента. Он выполняет ту же роль, что и `componentDidMount`, `componentDidUpdate` и `componentWillUnmount` в React-классах, объединив их в единый API.

Иногда мы хотим выполнить дополнительный код после того, как React обновил DOM. Сетевые запросы, изменения DOM вручную, логирование — всё это примеры эффектов, которые не требуют сброса.

~~~
function Example() {
  const [count, setCount] = useState(0);

  // По принципу componentDidMount и componentDidUpdate:
  useEffect(() => {
    // Обновляем заголовок документа, используя API браузера
    document.title = `Вы нажали ${count} раз`;
  });

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми на меня
      </button>
    </div>
  );
}
~~~

Поскольку эффекты объявляются внутри компонента, у них есть доступ к его пропсам и состоянию.

Этот же пример с использованием классов:

~~~
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `Вы нажали ${this.state.count} раз`;
  }

  componentDidUpdate() {
    document.title = `Вы нажали ${this.state.count} раз`;
  }

  render() {
    return (
      <div>
        <p>Вы нажали {this.state.count} раз</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Нажми на меня
        </button>
      </div>
    );
  }
}
~~~

Обратите внимание, что нам приходится дублировать наш код между классовыми методами жизненного цикла.

При необходимости вы можете вернуть из эффекта функцию, которая указывает эффекту, как выполнить за собой «сброс». Иногда очень важно выполнять сброс, чтобы не случилось утечек памяти!

~~~
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
});
~~~

В этом примере, React будет отписываться от нашего `ChatAPI` перед тем, как компонент размонтируется и перед тем, как перезапустить эффект при повторном рендере. В React-классе, вы, как правило, оформили бы подписку в `componentDidMount` и отменили бы её в `componentWillUnmount`. Здесь сброс эффекта происходит после каждого последующего рендера.

Эта логика по умолчанию гарантирует согласованность выполняемых нами действий и исключает баги, распространённые в классовых компонентах из-за упущенной логики обновления.

~~~
// Монтируем с пропсами { friend: { id: 100 } }
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);    // Выполняем первый эффект
// Обновляем с пропсами { friend: { id: 200 } }
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Сбрасываем предыдущий эффект
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);    // Выполняем следующий эффект
// Обновляем с пропсами { friend: { id: 300 } }
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Сбрасываем предыдущий эффект
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);    // Выполняем следующий эффект
// Размонтируем
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Сбрасываем последний эффект
~~~

Почему же мы вызываем `useEffect` непосредственно внутри компонента? Это даёт нам доступ к переменной состояния `count` (или любым другим пропсам) прямиком из эффекта. Нам не нужен специальный API для доступа к этой переменной — она уже находится у нас в области видимости функции.

Функция, которую мы передаём в `useEffect`, будет меняться при каждом рендере. Каждый раз при повторном рендере, мы ставим в очередь новый эффект, который заменяет предыдущий, то есть каждый эффект «принадлежит» определённому рендеру.

По умолчанию, React запускает эффекты после каждого рендера, включая первый рендер. React гарантирует, что он запустит эффект только после того, как DOM уже обновился.

### Преимущества `useEffect`:

* В отличие от `componentDidMount` или `componentDidUpdate`, эффекты, запланированные с помощью `useEffect`, не блокируют браузер при попытке обновить экран.
* В отличие от хуков, классовые методы жизненного цикла часто содержат логику, которая никак между собой не связана, в то время как связанная логика, разбивается на несколько методов. Это потому, что мы не можем использовать классовые методы жизненного цикла более одного раза, а `useEffect` можем. С помощью хуков, мы можем разделить наш код основываясь на том, что он делает, а не по принципам методов жизненного цикла.

### Советы:

* Используйте разные хуки для разных задач.
* Оптимизация производительности за счёт пропуска эффектов.

В некоторых случаях сброс или выполнение эффекта при каждом рендере может вызвать проблему с производительностью. В классовых компонентах, мы можем решить это используя дополнительное сравнение `prevProps` или `prevState` внутри `componentDidUpdate`:

~~~
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `Вы нажали ${this.state.count} раз`;
  }
}
~~~

Чтобы сделать это с хуком, передайте массив в `useEffect` вторым необязательным аргументом.

~~~
useEffect(() => {
  document.title = `Вы нажали ${count} раз`;
}, [count]); // Перезапускать эффект только если count поменялся
~~~

Здесь `useEffect` сработает только если `count` поменялся и при монтировании. Если поменяются другие переменные состояния, данный хук не сработает.

Также эффект не сработает, если присвоить `count` то же самое значение, так как React использует алгоритм сравнения `Object.is`.

Если у вас будет несколько элементов в массиве, React будет выполнять наш эффект, в том случае, когда хотя бы один из них будет отличаться. В качестве второго аргумента (массива) должны быть пропсы или переменные состояния. Это также работает для эффектов с этапом сброса.

Если вы хотите запустить эффект и сбросить его только один раз (при монтировании и размонтировании), вы можете передать пустой массив (`[]`) вторым аргументом. React посчитает, что ваш эффект не зависит от каких-либо значений из пропсов или состояния и поэтому не будет выполнять повторных запусков эффекта. Если вы передадите пустой массив (`[]`), пропсы и состояние внутри эффекта всегда будут иметь значения, присвоенные им изначально.

### Чем эффекты отличаются от событий?

Эффекты позволяют указать побочные эффекты, вызванные самим рендерингом, а не конкретным событием. Отправка сообщения в чат является событием , потому что оно напрямую вызвано нажатием пользователем определенной кнопки. Однако установка соединения с сервером является Эффектом , поскольку она должна происходить независимо от того, какое взаимодействие вызвало появление компонента.

Эффекты запускаются в конце фазы `commit` после обновления экрана. Это хорошее время для синхронизации компонентов React с какой-либо внешней системой (например, сетью или сторонней библиотекой).

### Пример с `video`

~~~
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }: { src: string; isPlaying: boolean }) {
  const ref = useRef<HTMLVideoElement>(null);

  useEffect(() => {
    if (isPlaying) {
      console.log("Calling video.play()");
      ref.current!.play();
    } else {
      console.log("Calling video.pause()");
      ref.current!.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState("");
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? "Pause" : "Play"}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
~~~

У вас мог бы возникнуть соблазн вызвать `play()` или `pause()` во время рендеринга, но это неправильно. В React рендеринг должен быть чистым вычислением JSX и не должен содержать побочных эффектов, таких как изменение DOM. И была бы ошибка `Cannot read properties of null (reading 'pause')`.

Оборачивая обновление DOM в эффект, вы позволяете React сначала обновить экран. Затем запускается ваш Эффект.

Указание `[isPlaying]` в качестве массива зависимостей сообщает React, что он должен пропустить повторный запуск вашего эффекта, если `isPlaying` такой же, как и во время предыдущего рендеринга.

Массив зависимостей может содержать несколько зависимостей. React пропустит повторный запуск Эффекта только в том случае, если все указанные вами зависимости имеют точно такие же значения, как и во время предыдущего рендеринга. React сравнивает значения зависимостей, используя функцию сравнения `Object.is`.

Обратите внимание, что вы не можете «выбрать» свои зависимости. Вы получите ошибку `lint`, если указанные вами зависимости не соответствуют ожиданиям React на основе кода внутри вашего эффекта. Это помогает обнаружить множество ошибок в вашем коде.

### Ловушка

По умолчанию эффекты запускаются после каждого рендеринга. Вот почему такой код создаст бесконечный цикл:

~~~
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
~~~

Эффекты запускаются в результате рендеринга. Настройка состояния запускает рендеринг.

### Как справиться с двойным срабатыванием Эффекта в разработке?

React намеренно перемонтирует ваши компоненты в процессе разработки, чтобы найти ошибки, как в последнем примере. Правильный вопрос не «как один раз запустить Эффект», а «как исправить мой Эффект, чтобы он работал после перемонтирования».

Обычно ответ заключается в реализации функции очистки. Функция очистки должна остановить или отменить действие Эффекта.

Иногда вам нужно добавить виджеты пользовательского интерфейса, которые не написаны для React. Например, допустим, вы добавляете компонент карты на свою страницу.

~~~
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
~~~

Обратите внимание, что в этом случае нет необходимости в очистке. В процессе разработки React дважды вызовет Effect, но это не проблема, потому что вызов `setZoomLevel` дважды с одним и тем же значением ничего не делает.

Некоторые API могут не позволять вызывать их дважды подряд. Например, метод `showModal` встроенного элемента `<dialog>` вызывает исключение, если вы вызываете его дважды. Реализуйте функцию очистки и закройте диалоговое окно:

~~~
чистки и закройте диалоговое окно:

useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
~~~

В процессе разработки ваш Эффект будет вызывать `showModal()`, затем немедленно `close()`, а затем снова `showModal()`. Это имеет такое же видимое для пользователя поведение, как однократный вызов `showModal()`, как вы могли бы видеть в `production`.

Если ваш Эффект подписывается на что-то, функция очистки должна отписаться:

~~~
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
~~~

Если ваш Эффект что-то анимирует, функция очистки должна сбросить анимацию до исходных значений:

~~~
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
~~~

Если ваш Эффект извлекает что-то, функция очистки должна либо прервать фетчинг либо проигнорировать ее результат:

~~~
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
~~~

Вы не можете «отменить» сетевой запрос, который уже был выполнен, но ваша функция очистки должна гарантировать, что фетчинг, который больше не актуален, не повлияет на ваше приложение. Если идентификатор пользователя изменится с «Алиса» на «Боб», очистка гарантирует, что ответ «Алиса» будет проигнорирован, даже если он придет после «Боба».

### Не эффект: инициализация приложения

Некоторая логика должна запускаться только один раз при запуске приложения. Вы можете поместить его вне своих компонентов:

~~~
if (typeof window !== 'undefined') { // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
~~~

Это гарантирует, что такая логика запускается только один раз после загрузки страницы браузером.

### Собираем все вместе

В этом примере используется `setTimeout`, чтобы запланировать вывод консоли с входным текстом через три секунды после запуска эффекта. Функция очистки отменяет отложенный тайм-аут.

~~~
import { useState, useEffect } from "react";

function Playground() {
  const [text, setText] = useState("a");

  useEffect(() => {
    function onTimeout() {
      console.log("⏰ " + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{" "}
        <input value={text} onChange={(e) => setText(e.target.value)} />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? "Unmount" : "Mount"} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
~~~

Сначала вы увидите три журнала: `Schedule "a" log`, `Cancel "a" log` и снова `Schedule "a" log`. Тремя секундами позже также появится лог с сообщением `a`. Как вы узнали ранее, дополнительная пара расписания/отмены связана с тем, что React повторно монтирует компонент после разработки, чтобы убедиться, что вы правильно реализовали очистку.

Теперь отредактируйте ввод, напишите `abc`.

~~~
🔵 Schedule "a" log
🟡 Cancel "a" log
🔵 Schedule "a" log
🟡 Cancel "a" log
🔵 Schedule "ab" log
🟡 Cancel "ab" log
🔵 Schedule "abc" log
⏰ abc
~~~

React всегда очищает эффект предыдущего рендера перед эффектом следующего рендера.

Введите что-нибудь в поле ввода, а затем сразу же нажмите `«Unmount the component»`. Обратите внимание, как размонтирование очищает эффект последнего рендера. Здесь он очищает последний тайм-аут, прежде чем он сможет запуститься.

Наконец, отредактируйте компонент выше и закомментируйте функцию очистки, чтобы тайм-ауты не отменялись. Попробуйте быстро набирать `abcde`.

Через три секунды вы должны увидеть последовательность логов (`a`, `ab`, `abc`, `abcd` и `abcde`), а не пять логов `abcde`. Каждый Эффект «захватывает» текстовое значение из соответствующего рендера. Неважно, что состояние текста изменилось: Эффект от рендера с `text = 'ab'` всегда будет видеть `'ab'`. Другими словами, Эффекты каждого рендера изолированы друг от друга.

### Обновление состояния на основе предыдущего состояния Эффекта

Если вы хотите обновить состояние на основе предыдущего состояния Эффекта, вы можете столкнуться с проблемой:

~~~
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // You want to increment the counter every second...
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... but specifying `count` as a dependency always resets the interval.
  // ...
}
~~~

Поскольку `count` является реактивным значением, его необходимо указать в списке зависимостей. Однако это приводит к очистке и настройке Эффекта каждый раз при изменении `count`. Это не идеально.

Чтобы исправить это, передайте `c => c + 1` в `setCount`.

### Удаление ненужных зависимостей

Если ваш Эффект зависит от объекта или функции, созданной во время рендеринга, он может запускаться слишком часто. Например, этот Эффект повторно подключается после каждого рендера, потому что объект `options` для каждого рендера разный:

~~~
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 This object is created from scratch on every re-render
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // It's used inside the Effect
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
}
~~~

Избегайте использования объекта, созданного во время рендеринга, в качестве зависимости. Вместо этого создайте объект внутри Эффекта.

### Настройка состояния при изменении пропсов

Иногда вам может понадобиться сбросить или настроить часть состояния при изменении пропсов, но не все.

~~~
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
~~~

Это не идеально. Каждый раз, когда элементы изменяются, список и его дочерние компоненты сначала отображаются с устаревшим значением выбора. Затем React обновит DOM и запустит эффекты. Наконец, вызов `setSelection(null)` вызовет еще один повторный рендеринг списка и его дочерних компонентов, снова перезапустив весь этот процесс.

Вместо сохранения (и сброса) выбранного элемента вы можете сохранить идентификатор выбранного элемента:

~~~
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
~~~

Оператор `??` возвращает первый аргумент, если он не `null`/`undefined`, иначе второй.

### Передача данных родителю

Этот компонент `Child` извлекает некоторые данные, а затем передает их компоненту `Parent` в эффекте:

~~~
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
~~~

Когда дочерние компоненты обновляют состояние своих родительских компонентов в Эффектах, поток данных становится очень трудно отследить. Поскольку и дочернему, и родительскому компонентам нужны одни и те же данные, позвольте родительскому компоненту получить эти данные и вместо этого передать их дочернему элементу:

~~~
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
~~~

Это проще и обеспечивает предсказуемость потока данных: данные перетекают от родителя к дочернему.

### Получение данных

Многие приложения используют Эффекты для начала фетчинга данных. Довольно часто эффект выборки данных записывается следующим образом:

~~~
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
~~~

Однако в приведенном выше коде есть ошибка. Представьте, что вы быстро печатаете `"hello"`. Затем `query` изменится с `"h"`, на `"he"`, `"hel"`, `"hell"` и `"hello"`. Это запустит отдельные фетчинги, но нет гарантии, в каком порядке будут поступать ответы.

Чтобы исправить состояние гонки, вам нужно добавить функцию очистки, чтобы игнорировать устаревшие ответы:

~~~
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
~~~

Это гарантирует, что когда ваш эффект будет получать данные, все ответы, кроме последнего запрошенного, будут проигнорированы.

Если вы хотели бы сделать выборку данных из Effects более эргономичной, рассмотрите возможность извлечения логики фетчинга в пользовательский хук, как в этом примере:

~~~
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
~~~
