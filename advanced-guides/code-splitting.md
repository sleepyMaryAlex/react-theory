# ?Code-Splitting

_Разделение кода_ — это возможность, поддерживаемая такими бандлерами как Webpack и другие, которая может создавать несколько бандлов и загружать их по мере необходимости.

Хоть вы и не уменьшите общий объём кода вашего приложения, но избежите загрузки кода, который может никогда не понадобиться пользователю и уменьшите объём кода, необходимый для начальной загрузки.

### `import()`

Лучший способ внедрить разделение кода в приложение — использовать синтаксис динамического импорта: `import()`.

До:
~~~
import { add } from './math';
console.log(add(16, 26));
~~~

После:
~~~
import("./math").then(math => {
  console.log(math.add(16, 26));
});
~~~

### `React.lazy`

Функция `React.lazy` позволяет рендерить динамический импорт как обычный компонент.

~~~
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
// вместо import OtherComponent from './OtherComponent';

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Загрузка...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
~~~

В этом примере код не будет загружен, пока вы не попытаетесь отобразить его. 

Компонент с ленивой загрузкой должен рендериться внутри компонента `Suspense`, который позволяет нам показать запасное содержимое (например, индикатор загрузки) пока происходит загрузка ленивого компонента. Можно обернуть несколько ленивых компонентов одним компонентом `Suspense`.

`React.lazy` в настоящее время поддерживает только экспорт по умолчанию.

### Avoiding fallbacks

Рассмотрим переключатель вкладок:

~~~
import React, { Suspense } from 'react';
import Tabs from './Tabs';

const Comments = React.lazy(() => import('./Comments'));
const Photos = React.lazy(() => import('./Photos'));

function MyComponent() {
  const [tab, setTab] = React.useState<string>('photos');

  function handleTabSelect(tab: string) {
    setTab(tab);
  };

  return (
    <div>
      <Tabs onTabSelect={handleTabSelect} />
      <Suspense fallback={<div>Загрузка...</div>}>
        {tab === 'photos' ? <Photos /> : <Comments />}
      </Suspense>
    </div>
  );
}
~~~

В этом примере, если вкладка изменяется с `'photos'` на `'comments'`, `Comments` приостанавливается, поэтому пользователь увидит мерцание. Это имеет смысл, потому что пользователь больше не хочет видеть `Photos`, `Comments` компонент не готов что-либо отображать, а React нужно поддерживать согласованность взаимодействия с пользователем, поэтому у него нет другого выбора, кроме как показать `Glimmer`.

Однако иногда такой пользовательский опыт нежелателен. В частности, иногда лучше показать «старый» UI, пока готовится новый UI. Вы можете использовать новый `startTransition` API, чтобы заставить React сделать это.

Этот метод предназначен для использования, когда `React.useTransition` недоступен.

~~~
function handleTabSelect(tab: string) {
  startTransition(() => {
    setTab(tab);
  });
}
~~~

Здесь вы сообщаете React, что настройка вкладки `'comments'` не является срочным обновлением, а является [переходом](https://ru.legacy.reactjs.org/docs/react-api.html#transitions), который может занять некоторое время.

### Error boundaries

Если какой-то модуль не загружается (например, из-за сбоя сети), это вызовет ошибку. Ошибка JavaScript где-то в коде UI не должна прерывать работу всего приложения.

_Предохранители_ — это компоненты React, которые отлавливают ошибки JavaScript в любом месте деревьев их дочерних компонентов, сохраняют их в журнале ошибок и выводят запасной UI вместо рухнувшего дерева компонентов.

Вы можете обрабатывать эти ошибки для улучшения пользовательского опыта с помощью _Error Boundaries_. После создания предохранителя, его можно использовать в любом месте над ленивыми компонентами для отображения состояния ошибки.

Классовый компонент является предохранителем, если он включает хотя бы один из следующих методов жизненного цикла: `static getDerivedStateFromError()` или `componentDidCatch()`. Используйте `static getDerivedStateFromError()` при рендеринге запасного UI в случае отлова ошибки. Используйте `componentDidCatch()` при написании кода для журналирования информации об отловленной ошибке.

~~~
import React, { Suspense } from 'react';
import ErrorBoundary from './ErrorBoundary';

const Comments = React.lazy(() => import('./Comments'));
const Photos = React.lazy(() => import('./Photos'));

function MyComponent() {
  return (
    <div>
    <ErrorBoundary>
      <Suspense fallback={<div>Загрузка...</div>}>
        <section>
          <Comments />
          <Photos />
        </section>
      </Suspense>
    </ErrorBoundary>
  </div>
  )
}

class ErrorBoundary extends React.Component<{children: ReactNode}, {hasError: boolean}> {
  constructor(props: {children: ReactNode}) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }
    return this.props.children;
  }
}
~~~

Предохранители работают как JavaScript-блоки `catch {}`, но только для компонентов. `try`/`catch` — отличная конструкция, но она работает исключительно в императивном коде. В то время, как компоненты React являются декларативными, указывая что должно быть отрендерено.

Только классовые компоненты могут выступать в роли предохранителей. На практике чаще всего целесообразным будет один раз описать предохранитель и дальше использовать его по всему приложению.

Предохранители отлавливают ошибки исключительно в своих дочерних компонентах. Предохранитель не сможет отловить ошибку внутри самого себя.

Где размещать предохранители? Например, вы можете защитить им навигационные (`route`) компоненты верхнего уровня, чтобы выводить пользователю сообщение «Что-то пошло не так», как это часто делают при обработке ошибок серверные фреймворки. Или вы можете охватить индивидуальными предохранителями отдельные виджеты, чтобы помешать им уронить всё приложение.

Пример:

App.tsx
~~~
function App() {
  return (
    <div>
      <ErrorBoundary>
        <p>These two counters are inside the same error boundary. If one crashes, the error boundary will replace both of them.</p>
        <BuggyCounter />
        <BuggyCounter />
      </ErrorBoundary>
      <hr />
      <p>These two counters are each inside of their own error boundary. So if one crashes, the other is not affected.</p>
      <ErrorBoundary><BuggyCounter /></ErrorBoundary>
      <ErrorBoundary><BuggyCounter /></ErrorBoundary>
    </div>
  );
}
~~~

BuggyCounter.tsx
~~~
function BuggyCounter() {
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter(counter + 1);
  }

  if (counter === 5) {
    throw new Error("I crashed!");
  }
  return <h1 onClick={handleClick}>{counter}</h1>;
}
~~~

ErrorBoundary.tsx
~~~
class ErrorBoundary extends React.Component<
  { children: ReactNode },
  { error: Error | null; errorInfo: { componentStack: string } | null }
> {
  constructor(props: { children: ReactNode }) {
    super(props);
    this.state = { error: null, errorInfo: null };
  }

  componentDidCatch(
    error: Error | null,
    errorInfo: { componentStack: string }
  ) {
    this.setState({
      error: error,
      errorInfo: errorInfo,
    });
  }

  render() {
    if (this.state.errorInfo) {
      return (
        <div>
          <h2>Something went wrong.</h2>
          <details>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }
    return this.props.children;
  }
}
~~~

Предохранители не отлавливают ошибки, произошедшие в обработчиках событий. Чтобы отловить ошибку в обработчике событий, пользуйтесь обычной JavaScript-конструкцией `try`/`catch`.

### Route-based code splitting

Решение о том, где в вашем приложении ввести разделение кода, может быть непростым. Часто таким удобным местом оказываются маршруты. Вот пример того, как организовать разделение кода на основе маршрутов с помощью `React.lazy` и таких библиотек как `React Router`:

~~~
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  </Router>
);
~~~
