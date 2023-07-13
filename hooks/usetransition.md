# ?useTransition

`useTransition` - это React Hook, который позволяет вам обновлять состояние, не блокируя пользовательский интерфейс.

`const [isPending, startTransition] = useTransition()`

`useTransition` не принимает никаких параметров.

`useTransition` возвращает массив ровно из двух элементов:

* Флаг `isPending`, указывающий, есть ли ожидающий переход.
* Функция `startTransition`, позволяющая отметить обновление состояния как переход.

### Функция `startTransition`

~~~
startTransition(() => {
  setTab(nextTab);
});
~~~

Параметры:

* `scope`: функция, которая обновляет некоторое состояние, вызывая одну или несколько функций `set`. React немедленно вызывает `scope` без параметров и помечает все обновления состояния, запланированные синхронно во время вызова функции `scope`, как переходы. Они будут неблокирующими и не будут отображать нежелательные индикаторы загрузки.

`startTransition` ничего не возвращает.

> `startTransition` очень похож на `useTransition`, за исключением того, что он не предоставляет флаг `isPending` для отслеживания того, продолжается ли переход. Вы можете вызвать `startTransition`, когда `useTransition` недоступен.

### Применение

Переходы позволяют сохранять отзывчивость обновлений пользовательского интерфейса даже на медленных устройствах.

Например, если пользователь щелкает вкладку, но затем передумал и щелкнул другую вкладку, он может сделать это, не дожидаясь завершения первого повторного рендеринга.

В этом примере вкладка `«Posts»` искусственно замедляется , чтобы отрисовка занимала не менее секунды.

Нажмите `«Posts»`, а затем сразу нажмите `«Contact»`. Обратите внимание, что это прерывает медленный рендеринг `«Posts»`. Вкладка `«Contact»` отображается сразу. Поскольку это обновление состояния помечено как переход, медленный повторный рендеринг не приводит к зависанию пользовательского интерфейса.

App.tsx
~~~
import { useState, useTransition } from "react";
import TabButton from "./TabButton";
import AboutTab from "./AboutTab";
import PostsTab from "./PostsTab";
import ContactTab from "./ContactTab";

export default function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState("about");

  function selectTab(nextTab: string) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <>
      <TabButton isActive={tab === "about"} onClick={() => selectTab("about")}>
        About
      </TabButton>
      <TabButton isActive={tab === "posts"} onClick={() => selectTab("posts")}>
        Posts (slow)
      </TabButton>
      <TabButton
        isActive={tab === "contact"}
        onClick={() => selectTab("contact")}
      >
        Contact
      </TabButton>
      <hr />
      {tab === "about" && <AboutTab />}
      {tab === "posts" && <PostsTab />}
      {tab === "contact" && <ContactTab />}
    </>
  );
}
~~~

TabButton.tsx
~~~
import { ReactNode, useTransition } from "react";

export default function TabButton({
  children,
  isActive,
  onClick,
}: {
  children: ReactNode;
  isActive: boolean;
  onClick: () => void;
}) {
  if (isActive) {
    return <b>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        onClick();
      }}
    >
      {children}
    </button>
  );
}
~~~

AboutTab.tsx
~~~
export default function AboutTab() {
  return <p>Welcome to my profile!</p>;
}
~~~

ContactTab.tsx
~~~
export default function ContactTab() {
  return (
    <>
      <p>You can find me online here:</p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
~~~

PostsTab.tsx
~~~
import { memo } from "react";

const PostsTab = memo(function PostsTab() {
  // Log once. The actual slowdown is inside SlowPost.
  console.log("[ARTIFICIALLY SLOW] Rendering 500 <SlowPost />");

  let items = [];
  for (let i = 0; i < 500; i++) {
    items.push(<SlowPost key={i} index={i} />);
  }
  return <ul className="items">{items}</ul>;
});

function SlowPost({ index }: { index: number }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Do nothing for 1 ms per item to emulate extremely slow code
  }

  return <li className="item">Post #{index + 1}</li>;
}

export default PostsTab;
~~~

Вы можете использовать `isPending`, возвращаемое `useTransition`, чтобы указать пользователю, что переход выполняется.

~~~
if (isPending) {
  return <b className="pending">Pending...</b>;
}
~~~
