# ?useSyncExternalStore

`useSyncExternalStore` - это React Hook, который позволяет вам подписаться на внешнее хранилище.

`const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

Параметры:

* `subscribe`: функция, которая принимает один аргумент `callback` и подписывается на хранилище. Когда хранилище изменяется, оно должно вызывать предоставленный `callback`. Это приведет к повторному рендерингу компонента. Функция подписки должна возвращать функцию, очищающую подписку.

* `getSnapshot`: функция, которая возвращает моментальный снимок данных в хранилище, который необходим компоненту. Пока хранилище не изменилось, повторные вызовы `getSnapshot` должны возвращать одно и то же значение. Если хранилище изменяется и возвращаемое значение отличается (по сравнению с `Object.is`), React повторно отображает компонент.

* необязательный `getServerSnapshot`: функция, которая возвращает начальный снимок данных в хранилище. Он будет использоваться только во время серверного рендеринга и во время гидратации серверного контента на клиенте. Моментальный снимок сервера должен быть одним и тем же между клиентом и сервером, и обычно сериализуется и передается от сервера к клиенту. Если вы опустите этот аргумент, рендеринг компонента на сервере вызовет ошибку.

Возвращает текущий снимок хранилища, который вы можете использовать в своей логике рендеринга.

### Применение

Большинство ваших компонентов React будут считывать данные только из своих свойств,  состояния и контекста. Однако иногда компоненту необходимо прочитать некоторые данные из какого-то хранилища за пределами React, которые со временем меняются. Это включает в себя:

* Сторонние библиотеки управления состоянием, которые хранят состояние вне React.
* API-интерфейсы браузера, предоставляющие изменяемое значение и события для подписки на его изменения.

Допустим, вы хотите подписаться на какое-то значение, предоставляемое браузером, которое меняется со временем. Например, предположим, что вы хотите, чтобы ваш компонент отображал, активно ли сетевое подключение. Браузер предоставляет эту информацию через свойство с именем `navigator.onLine`.

Это значение может измениться без ведома React, поэтому вы должны прочитать его с `useSyncExternalStore`.

Обычно вы не будете писать `useSyncExternalStore` непосредственно в своих компонентах. Вместо этого вы обычно вызываете его из своего собственного хука. Это позволяет использовать одно и то же внешнее хранилище из разных компонентов.

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

useOnlineStatus.ts
~~~
import { useSyncExternalStore } from "react";

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback: () => void) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}
~~~

Теперь React умеет читать значение из внешнего API `navigator.onLine` и подписываться на его изменения.
