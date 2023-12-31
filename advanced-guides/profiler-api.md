# ?Profiler API

_Profiler_ измеряет то, как часто рендерится React-приложение и какова «стоимость» этого.

Его задача — помочь найти медленные части приложения, которые можно оптимизировать (например, через мемоизацию).

Если ваше приложение использует React 16.5 или выше, оно будет поддерживать новый подключаемый модуль `DevTools Profiler`.

Расширение `React DevTools` должно быть установлено в вашем браузере.

### Вкладка Profiler

Вкладка `Profiler` очень удобна, так как обеспечивает визуальное представление времени рендеринга и позволяет перемещаться по коммитам. В результате это позволяет быстрее анализировать и исследовать проблемы с производительностью.

`React Profiler` предоставляет несколько типов представления диаграмм для данных о производительности.

Некоторые из них:

1. __Flamegraph chart.__ Это представление представляет состояние вашего приложения, когда фиксируется обновление. Каждая полоса представляет компонент, а длина полосы показывает, сколько времени потребовалось для визуализации компонента. Нажав на каждую полосу, вы можете получить конкретную информацию о рендеринге для этого компонента для выбранного коммита на правой панели сведений.

Используя цвета столбцов, вы можете получить представление о времени, которое потребовалось для рендеринга каждого компонента. Общие цвета следующие.

* __Серый__: компонент не отобразился во время фиксации.
* __Сине-зеленый__: рендеринг компонента занял меньше времени.
* __Желто-зеленый__: рендеринг компонента занял больше времени.
* __Желтый__: рендеринг компонента занял больше всего времени.

2. __Ranked chart.__ В этом представлении отображается порядок компонентов в зависимости от времени их визуализации. Компоненты, которые заняли больше времени, будут наверху.

Измерение производительности вашего приложения в режиме разработки может дать вам ложные результаты. Чтобы получить точные измерения, вам необходимо профилировать ваше приложение в рабочем режиме, так как это то, с чем столкнутся пользователи.

Чтобы запустить проект после сборки, нужно указать `homepage` в `package.json`. Создать сборку профилирования можно, указав дополнительный `--profile` флаг:

~~~
yarn build --profile
~~~

Расширение `DevTools` — не единственный способ измерения производительности вашего приложения. Вы также можете использовать `<Profiler>` компонент React (`React Profiler API`) в своем коде для измерения производительности.

### React Profiler API

Оберните дерево компонентов в `<Profiler>`, чтобы измерить его производительность рендеринга.

~~~
function App() {
  const [clicks, setClicks] = useState(0);

  function handleClick() {
    setClicks(clicks + 1);
  }

  function onRenderCallback(
    id: string,
    phase: "mount" | "update",
    actualDuration: number,
    baseDuration: number,
    startTime: number,
    commitTime: number,
    interactions: Set<any>
  ) {
    // Обработка или логирование результатов...
    console.log(id); // App
    console.log(phase); // update / mount
    console.log(actualDuration); // 0.10000002384185791 - время, затраченное на рендер зафиксированного обновления
    console.log(baseDuration); // предполагаемое время рендера всего поддерева без кеширования
    console.log(startTime); // когда React начал рендерить это обновление
    console.log(commitTime); // когда React зафиксировал это обновление
    console.log(interactions); // Множество «взаимодействий» для данного обновления
  }

  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <p>Количество кликов: {clicks}</p>
      <button onClick={handleClick}>Кликните</button>
    </Profiler>
  );
}
~~~

`id`: строка, идентифицирующая часть пользовательского интерфейса, которую вы измеряете.
`onRender`: обратный вызов `onRender`, который React вызывает каждый раз, когда компоненты в профилированном дереве обновляются. Он получает информацию о том, что было отрендерено и сколько времени это заняло.

Для замера разных частей приложения могут быть использованы несколько компонентов `Profiler`. Также `Profiler` может быть вложенным с целью замера разных компонентов внутри поддерева.

Несмотря на то `<Profiler>`, что это легкий компонент, его следует использовать только при необходимости. Каждое использование увеличивает нагрузку на ЦП и память приложения.
