# ?Removing effect dependencies

Иногда вы хотите, чтобы ваш Эффект «реагировал» на определенное значение, но это значение меняется чаще, чем вам хотелось бы, и может не отражать реальных изменений с точки зрения пользователя.

Например, предположим, что вы создаете объект `options` в теле вашего компонента, а затем читаете этот объект изнутри вашего Эффекта:

App.tsx
~~~
import { useState, useEffect } from 'react';
import { createConnection } from './chat';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }: { roomId: string }) {
  const [message, setMessage] = useState('');

  // Temporarily disable the linter to demonstrate the problem
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
~~~

chat.ts
~~~
export function createConnection({ serverUrl, roomId }: { serverUrl: string, roomId: string }) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
~~~

Каждый раз, когда вы обновляете `message`, ваш компонент перерисовывается. Когда ваш компонент перерисовывается, код внутри него снова запускается с нуля.

Новый объект `options` создается с нуля при каждом повторном рендеринге компонента `ChatRoom`. React видит, что объект `options` отличается от объекта `options`, созданного во время последнего рендеринга. Вот почему он повторно синхронизирует ваш Эффект (который зависит от `options`), и чат повторно подключается по мере того, как вы печатаете.

Эта проблема затрагивает только объекты и функции. В JavaScript каждый вновь созданный объект и функция считаются отличными от всех остальных. Неважно, что содержимое внутри них может быть одинаковым!

Зависимости объектов и функций могут привести к повторной синхронизации вашего Эффекта чаще, чем вам нужно.

Вот почему, когда это возможно, вы должны стараться избегать объектов и функций в качестве зависимостей вашего Эффекта. Вместо этого попробуйте переместить их за пределы компонента, внутрь Эффекта или извлечь из них примитивные значения.

### Чтение примитивных значений из объектов

Иногда вы можете получить объект из пропсов:

~~~
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
}
~~~

Риск здесь заключается в том, что родительский компонент создаст объект во время рендеринга:

~~~
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
~~~


Это приведет к тому, что ваш эффект будет повторно подключаться каждый раз, когда родительский компонент повторно рендерится. Чтобы это исправить, считывайте информацию с объекта вне Эффекта и избегайте объектов и функций в зависимостях:

~~~
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
}
~~~

Логика становится немного повторяющейся (вы считываете некоторые значения из объекта вне Эффекта, а затем создаете объект с теми же значениями внутри Эффекта). Но это делает очень явным, от какой информации на самом деле зависит ваш Эффект. Если объект непреднамеренно воссоздается родительским компонентом, чат не будет повторно подключаться. Однако, если `options.roomId` или `options.serverUrl` действительно отличаются, чат снова подключится.
