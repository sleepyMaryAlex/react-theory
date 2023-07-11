# ?useId

`useId` — это React Hook для создания уникальных идентификаторов, которые можно передать атрибутам доступности.

`const id = useId()`

`useId` не принимает никаких параметров.

`useId` возвращает уникальную строку идентификатора, связанную с этим конкретным вызовом `useId` в этом конкретном компоненте.

`useId` не следует использовать для генерации ключей в списке. Ключи должны быть сгенерированы из ваших данных.

### Применение

Такие атрибуты доступности HTML, как `aria-describedby` позволяют указать, что два тега связаны друг с другом. Например, вы можете указать, что элемент (например, `input`) описывается другим элементом (например, `p`).

В обычном HTML вы бы написали это так:

~~~
<label>
  Password:
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  The password should contain at least 18 characters
</p>
~~~

Однако такое жесткое кодирование идентификаторов не является хорошей практикой в ​​React. Компонент может отображаться на странице более одного раза, но идентификаторы должны быть уникальными! Вместо жесткого кодирования идентификатора создайте уникальный идентификатор с помощью `useId`:

~~~
function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}
~~~

Теперь, даже если `PasswordField` появляется на экране несколько раз, сгенерированные идентификаторы не будут конфликтовать.

### Создание идентификаторов для нескольких связанных элементов

Если вам нужно присвоить идентификаторы нескольким связанным элементам, вы можете вызвать `useId`, чтобы сгенерировать для них общий префикс:

~~~
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>First Name:</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>Last Name:</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
~~~

Это позволяет избежать вызова `useId` для каждого отдельного элемента, которому требуется уникальный идентификатор.
