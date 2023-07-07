# ?Custom hooks

Иногда нужно повторно использовать одинаковую логику состояния в нескольких компонентах. Традиционно использовались два подхода: компоненты высшего порядка и рендер-пропсы. С помощью пользовательских хуков эта задача решается без добавления ненужных компонентов в ваше дерево.

Давайте извлечём логику в пользовательский хук `useFriendStatus`.

~~~
function useCountColor(count: number) {
  const [isRed, setIsRed] = useState<boolean>(false);

  useEffect(() => {
    count < 3 ? setIsRed(false) : setIsRed(true);
  }, [count]);

  return isRed;
}
~~~

Теперь мы можем использовать этот хук в нескольких компонентах:

~~~
function Counter() {
  const [count, setCount] = useState(0);
  const isRed = useCountColor(count);

  const doubleIncreaseHandler = () => {
    setCount(count + 1)
  }

  return (
    <div>
      <button onClick={doubleIncreaseHandler}>Increase</button>
      <div style={{color: isRed ? 'red' : 'green'}}>Count: {count}</div>
    </div>
  )
}
~~~

Состояния каждого компонента никаким образом не зависят друг от друга. Хуки — это способ использовать повторно логику состояния, а не само состояние. Более того, каждое обращение к хуку обеспечивает совершенно изолированное состояние. Вы даже можете использовать один и тот же хук несколько раз в одном компоненте.

_Пользовательские хуки_ — это в большей степени соглашение, чем дополнение. Если имя функции начинается с `use` и она вызывает другие хуки, мы расцениваем это как пользовательский хук. Если вы будете придерживаться соглашения `useSomething` при именовании хуков, это позволит нашему плагину для линтера найти баги в коде, который использует хуки.

### Пользовательские хуки: совместное использование логики между компонентами

Представьте, что вы разрабатываете приложение, которое сильно зависит от сети (как и большинство приложений). Вы хотите предупредить пользователя, если его сетевое соединение случайно отключилось во время использования вашего приложения.

Вы можете начать с чего-то вроде этого:

~~~
import { useState, useEffect } from "react";

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

~~~

Попробуйте включить и выключить сеть и обратите внимание, как `StatusBar` обновляется в ответ на ваши действия.

Теперь представьте, что вы хотите использовать ту же логику в другом компоненте. Вы хотите реализовать кнопку `«Save progress»`, которая будет отключена и будет отображать `«Reconnecting...»` вместо `«Save progress»`, когда сеть отключена.

~~~
import { useState, useEffect } from "react";

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}
~~~

Эти два компонента работают нормально, но оба этих компонента можно было бы упростить и убрать дублирование между ними.

### Извлечение собственного пользовательского хука из компонента

Объявите `useOnlineStatus` и переместите в нее весь дублированный код из компонентов, которые вы написали ранее:

useOnlineStatus.ts
~~~
import { useState, useEffect } from "react";

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
~~~

App.tsx
~~~
import { useOnlineStatus } from "./useOnlineStatus";

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
~~~

Теперь в ваших компонентах меньше повторяющейся логики. Что еще более важно, код внутри них описывает, что они хотят делать (использовать онлайн-статус!), а не как это делать (подписываясь на события браузера).

### Имена хуков всегда начинаются с `use`

Вы должны следовать соглашениям об именах:

* Имена компонентов React должны начинаться с заглавной буквы, например `StatusBar` и `SaveButton`. Компоненты React также должны возвращать то, что React умеет отображать, например фрагмент JSX.
* Имена хуков должны начинаться с `use`, за которым следует заглавная буква, например `useState` (встроенный) или `useOnlineStatus` (пользовательский, как ранее на странице). Хуки могут возвращать произвольные значения.

Это соглашение гарантирует, что вы всегда можете посмотреть на компонент и узнать, где его состояние, эффекты и другие функции React могут «спрятаться». Например, если вы видите вызов функции `getColor()` внутри вашего компонента, вы можете быть уверены, что он не может содержать внутри состояние React, потому что его имя не начинается с `use`.

Если ваш линтер настроен для React, он будет применять это соглашение об именах. Прокрутите вверх до песочницы выше и переименуйте `useOnlineStatus` в `getOnlineStatus`. Обратите внимание, что линтер больше не позволит вам вызывать `useState` или `useEffect` внутри него. Только хуки и компоненты могут вызывать другие хуки!

### Пользовательские хуки позволяют вам делиться логикой с отслеживанием состояния, а не самим состоянием

В предыдущем примере, когда вы включали и выключали сеть, оба компонента обновлялись вместе. Однако неправильно думать, что между ними используется одна переменная состояния `isOnline`.

Это две полностью независимые переменные состояния и Эффекты! У них оказалось одно и то же значение в одно и то же время, потому что вы синхронизировали их с одним и тем же внешним значением (независимо от того, включена ли сеть).

### Передача реактивных значений

Код внутри ваших пользовательских хуков будет повторно запускаться при каждом повторном рендеринге вашего компонента. Вот почему, как и компоненты, пользовательские хуки должны быть чистыми. Думайте о пользовательском коде хуков как о части тела вашего компонента!

Пользовательский хук не обязательно должен что-то возвращать:

useChatRoom.js
~~~
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
~~~

Обратите внимание, как вы принимаете возвращаемое значение `useState` и передайте его как вход `useChatRoom`:

ChatRoom.js
~~~
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
}
~~~

Каждый раз, когда ваш компонент `ChatRoom` выполняет повторный рендеринг, он передает последний `roomId` и `serverUrl` вашему хуку. Вот почему ваш эффект повторно подключается к чату всякий раз, когда их значения отличаются после повторного рендеринга.

### Передача обработчиков событий в пользовательские хуки

По мере того, как вы начнете использовать useChatRoom в большем количестве компонентов, вы, возможно, захотите позволить компонентам настраивать его поведение. Например, в настоящее время логика того, что делать, когда приходит сообщение, жестко запрограммирована внутри хука:

~~~
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
~~~

Допустим, вы хотите переместить эту логику обратно в свой компонент:

~~~
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
}
~~~

Чтобы это работало, измените свой пользовательский хук, чтобы использовать `onReceiveMessage` в качестве одной из названных опций:

~~~
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onReceiveMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ All dependencies declared
}
~~~

Добавление зависимости от `onReceiveMessage` не идеально, потому что это приведет к повторному подключению чата каждый раз, когда компонент повторно отрисовывается. Оберните этот обработчик события в событие эффекта, чтобы удалить его из зависимостей:

~~~
import { useEffect, useEffectEvent } from 'react';
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
}
~~~

### Когда использовать пользовательские хуки

Вам не нужно извлекать пользовательский хук для каждого небольшого дублированного фрагмента кода. Некоторое дублирование допустимо. Например, извлечение хука `useFormInput` для переноса одного `useState`, вероятно, не нужно.

Однако всякий раз, когда вы пишете эффект, подумайте, не будет ли понятнее заключить его в пользовательский хук.

Например, рассмотрим компонент `ShippingForm`, который отображает два раскрывающихся списка: один показывает список городов, а другой показывает список областей в выбранном городе.

~~~
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // This Effect fetches cities for a country
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // This Effect fetches areas for the selected city
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);

  // ...
}
~~~

Хотя этот код довольно повторяющийся, правильно хранить эти Эффекты отдельно друг от друга. Они синхронизируют две разные вещи, поэтому их не следует объединять в один Эффект. Вместо этого вы можете упростить компонент `ShippingForm`, извлекая общую логику между ними в свой собственный хук `useData`:

~~~
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
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
    }
  }, [url]);
  return data;
}
~~~

Теперь вы можете заменить оба эффекта в `ShippingForm` вызовами `useData`:

~~~
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
}
~~~

Со временем большая часть эффектов вашего приложения будет находиться в пользовательских хуках.

### Пользовательские хуки помогут вам перейти на лучшие шаблоны

Вернемся к этому примеру:

App.tsx
~~~
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
~~~

useOnlineStatus.ts
~~~
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
~~~

В приведенном выше примере `useOnlineStatus` реализован с помощью пары `useState` и `useEffect`. Однако это не лучшее возможное решение. Существует ряд крайних случаев, которые он не рассматривает. Например, предполагается, что при монтировании компонента `isOnline` уже имеет значение `true`, но это может быть неверным, если сеть уже отключилась. Вы можете использовать API браузера `navigator.onLine`, чтобы проверить это, но его прямое использование не будет работать на сервере для создания исходного HTML. Короче говоря, этот код можно улучшить.

К счастью, React 18 включает специальный API под названием `useSyncExternalStore`, который решит все эти проблемы за вас. Вот как переписан ваш хук `useOnlineStatus`, чтобы использовать преимущества этого нового API:

App.tsx
~~~
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ Online" : "❌ Disconnected"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progress saved");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Save progress" : "Reconnecting..."}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
~~~

useOnlineStatus.ts
~~~
function subscribe(callback: () => void) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}
~~~

### Существует более одного способа сделать это

Допустим, вы хотите реализовать анимацию постепенного появления с нуля с помощью API браузера `requestAnimationFrame`.

Чтобы сделать компонент более читабельным, вы можете извлечь логику в пользовательский хук `useFadeIn`:

App.tsx
~~~
import { useState, useRef } from "react";
import { useFadeIn } from "./useFadeIn";

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>{show ? "Remove" : "Show"}</button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
~~~

useFadeIn.ts
~~~
import { useEffect } from "react";

export function useFadeIn(
  ref: { current: HTMLHeadingElement | null },
  duration: number
) {
  useEffect(() => {
    const node = ref!.current;

    let startTime: number | null = performance.now();
    let frameId: number | null = null;

    function onFrame(now: number) {
      const timePassed = now - startTime!;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // We still have more frames to paint
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress: number) {
      node!.style.opacity = progress.toString();
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId!);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, [ref, duration]);
}
~~~

Однако эту конкретную анимацию постепенного появления проще и эффективнее реализовать с помощью простой CSS-анимации:

App.tsx
~~~
import { useState } from "react";
import "./welcome.css";

function Welcome() {
  return <h1 className="welcome">Welcome</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>{show ? "Remove" : "Show"}</button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
~~~

welcome.css
~~~
.welcome {
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);

  animation: fadeIn 1000ms;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}
~~~

Иногда вам даже не нужен хук!
