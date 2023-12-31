# ?useDeferredValue

`useDeferredValue` — это React Hook, который позволяет вам отложить обновление части пользовательского интерфейса.

`const deferredValue = useDeferredValue(value)`

Параметры:

* `value`: значение, которое вы хотите отложить. Он может иметь любой тип.

Возвращает:

Во время первоначального рендеринга возвращаемое отложенное значение будет таким же, как и предоставленное вами значение. Во время обновлений React сначала попытается выполнить повторный рендеринг со старым значением, а затем попытается выполнить еще один повторный рендеринг в фоновом режиме с новым значением.

Предостережения:

* Значения, которые вы передаете в `useDeferredValue`, должны быть либо примитивными значениями (например, строками и числами), либо объектами, созданными вне рендеринга. Если вы создадите новый объект во время рендеринга и сразу же передадите его в `useDeferredValue`, он будет отличаться при каждом рендеринге, вызывая ненужную повторную визуализацию фона.

* Когда `useDeferredValue` получает другое значение (по сравнению с `Object.is`), в дополнение к текущему рендерингу (когда он все еще использует предыдущее значение), он планирует повторный рендеринг в фоновом режиме с новым значением. Фоновый повторный рендеринг можно прервать: если произойдет еще одно обновление значения, React перезапустит фоновый повторный рендеринг с нуля. Например, если пользователь вводит ввод быстрее, чем `chart`, получающая отложенное значение, может повторно отобразиться, `chart` будет повторно отображаться только после того, как пользователь перестанет печатать.

* `useDeferredValue` сам по себе не предотвращает дополнительные сетевые запросы.

* Не существует фиксированной задержки, вызванной самим `useDeferredValue`. Как только React завершит первоначальный повторный рендеринг, React немедленно начнет работать над фоновым повторным рендерингом с новым отложенным значением.

Вызовите `useDeferredValue` на верхнем уровне вашего компонента, чтобы получить отложенную версию значения.

### Отложенный повторный рендеринг списка

Представьте, что у вас есть текстовое поле и компонент (длинный список), который повторно отображается при каждом нажатии клавиши.

Чтобы набор текста не был дерганым, `useDeferredValue` устанавливает приоритет обновления ввода (который должен быть быстрым) над обновлением списка результатов (который может быть медленнее):

App.tsx
~~~
import { useState, useDeferredValue } from "react";
import SlowList from "./SlowList";

export default function App() {
  const [text, setText] = useState<string>("");
  const deferredText = useDeferredValue(text);

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
~~~

SlowList.tsx
~~~
import { memo } from "react";

const SlowList = memo(function SlowList({ text }: { text: string }) {
  // Log once. The actual slowdown is inside SlowItem.
  console.log(text);

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return <ul className="items">{items}</ul>;
});

function SlowItem({ text }: { text: string }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Do nothing for 1 ms per item to emulate extremely slow code
  }

  return <li className="item">Text: {text}</li>;
}

export default SlowList;
~~~

Это не ускоряет повторный рендеринг `SlowList`. Однако он сообщает React, что повторный рендеринг списка может быть лишен приоритета, чтобы он не блокировал нажатия клавиш. Список будет «отставать» от ввода, а затем «догонять». Как и раньше, React попытается обновить список как можно скорее, но не будет блокировать ввод текста пользователем.

Эта оптимизация требует, чтобы `SlowList` был обернут в `memo`. Это связано с тем, что всякий раз, когда текст изменяется, React должен иметь возможность быстро повторно отображать родительский компонент. Во время этого повторного рендеринга `deferredText` по-прежнему имеет свое предыдущее значение, поэтому `SlowList` в состоянии пропустить повторный рендеринг (его пропсы не изменились). Без `memo` его все равно пришлось бы перерендерить, что нарушило бы смысл оптимизации.
