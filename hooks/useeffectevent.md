# ?useEffectEvent

Представьте, что вы хотите показывать уведомление, когда пользователь подключается к чату. Вы читаете текущую тему (темную или светлую) из пропсов, чтобы вы могли показать уведомление в правильном цвете.

Однако `theme` это реактивное значение (оно может измениться в результате повторного рендеринга), и каждое реактивное значение, прочитанное Эффектом, должно быть объявлено как его зависимость.

App.tsx
~~~
import { useState, useEffect } from "react";
import { createConnection } from "./chat";
import { showNotification } from "./notifications";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId, theme }: { roomId: string, theme: string }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on("connected", () => {
      showNotification("Connected!", theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={(e) => setIsDark(e.target.checked)}
        />
        Use dark theme
      </label>
      <hr />
      <ChatRoom roomId={roomId} theme={isDark ? "dark" : "light"} />
    </>
  );
}
~~~

chat.ts
~~~
export function createConnection(serverUrl: string, roomId: string) {
  // A real implementation would actually connect to the server
  let connectedCallback: (() => void) | null = null;
  let timeout: ReturnType<typeof setTimeout>;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event: string, callback: () => void) {
      if (connectedCallback) {
        throw Error("Cannot add the handler twice.");
      }
      if (event !== "connected") {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    },
  };
}
~~~

notification.ts
~~~
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message: string, theme: string) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
~~~

При изменении `roomId` чат снова подключается, как и следовало ожидать. Но поскольку `theme` это тоже зависимость, чат также повторно подключается каждый раз, когда вы переключаетесь между темной и светлой темой. Это не здорово!

Другими словами, вы не хотите, чтобы строка `showNotification('Connected!', theme);` была реактивной, даже если она находится внутри Эффекта (который является реактивным).

Вам нужен способ отделить нереактивную логику от реактивного Эффекта.

Используйте специальный хук `useEffectEvent`, чтобы извлечь нереактивную логику из вашего Эффекта:

~~~
function ChatRoom({ roomId, theme }: { roomId: string, theme: string }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
~~~

Здесь `onConnected` называется событием эффекта. Это часть вашей логики Effect, но она ведет себя гораздо больше как обработчик событий. Логика внутри него не реактивна, и он всегда «видит» последние значения ваших свойств и состояния.

Это решает проблему. Обратите внимание, что вам нужно удалить `onConnected` из списка зависимостей вашего эффекта. События Effect не являются реактивными и должны быть исключены из зависимостей.

> `useEffectEvent` - экспериментальный API, который еще не выпущен в стабильной версии React.

Вы можете думать о событиях эффектов как об очень похожих на обработчики событий. Основное отличие состоит в том, что обработчики событий запускаются в ответ на действия пользователя, тогда как события эффектов запускаются вами из эффектов. События Эффектов позволяют «разорвать цепочку» между реактивностью Эффектов и кодом, который не должен быть реактивным.

### Чтение последних пропсов и состояния с помощью событий эффектов

Есть такой код:

~~~
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
~~~

Вам может быть интересно, можно ли вызвать `onVisit()` без аргументов и прочитать `url` внутри него:

~~~
const onVisit = useEffectEvent(() => {
  logVisit(url, numberOfItems);
});

useEffect(() => {
  onVisit();
}, [url]);
~~~

Это сработает, но лучше явно передать `url` событию эффекта. Передавая в `url` качестве аргумента вашему событию эффекта, вы говорите, что посещение страницы с другим `url` представляет собой отдельное «событие» с точки зрения пользователя.

Поскольку ваше событие Effect явно «запрашивает» `VisitUrl`, теперь вы не можете случайно удалить `url` из зависимостей Effect. Если вы удалите зависимость от `url` (что приведет к тому, что посещения разных страниц будут считаться за одно), линтер предупредит вас об этом. Вы хотите, чтобы `onVisit` реагировал на `url`, поэтому вместо того, чтобы читать `url` внутри (где он не будет реактивным), вы передаете его из вашего Эффекта.

Это становится особенно важным, если внутри Эффекта есть какая-то асинхронная логика:

~~~
const onVisit = useEffectEvent(visitedUrl => {
  logVisit(visitedUrl, numberOfItems);
});

useEffect(() => {
  setTimeout(() => {
    onVisit(url);
  }, 5000); // Delay logging visits
}, [url]);
~~~

Здесь `url` внутри `onVisit` соответствует последнему `url` (который мог уже измениться), но `VisitUrl` соответствует `url`, который первоначально вызвал запуск этого эффекта (и вызова `onVisit`).

### Ограничения событий эффектов

События эффектов очень ограничены в том, как вы можете их использовать:

* Вызывайте их только изнутри Effects.
* Никогда не передавайте их другим компонентам или хукам.

События эффектов — это нереактивные «части» вашего кода эффекта. Они должны быть рядом с Эффектом, использующим их.
