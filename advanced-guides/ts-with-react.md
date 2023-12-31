# ?TS with REACT

`Create React App` поддерживает TypeScript по умолчанию.

Чтобы создать новый проект с поддержкой TypeScript, используйте следующую команду:

~~~
npx create-react-app my-app --template typescript
~~~

В TypeScript лучше разделять файлы на два типа: `.tsx` для файлов, содержащих разметку JSX, и `.ts` для всего остального.

### Определения типов

Для анализа ошибок и выдачи всплывающих подсказок компилятор TypeScript использует файлы объявлений. Они содержат в себе всю информацию о типах, которые используются в конкретной библиотеке.

В свою очередь это позволяет нам использовать JavaScript-библиотеки в проекте совместно с TypeScript.

_DefinitelyTyped_ — это внушительный репозиторий файлов объявлений. Например, React устанавливается без собственного файла объявления — вместо этого мы устанавливаем его отдельно:

~~~
yarn add --dev @types/react
~~~

~~~
npm i --save-dev @types/react
~~~

Иногда пакет, который вы хотите использовать, не имеет ни собственного файла объявлений, ни соответствующего файла в репозитории `DefinitelyTyped`. В этом случае, мы можем объявить собственный локальный файл объявлений.

Для этого надо создать файл с расширением `d.ts` в `src/*`. Файл объявлений может выглядеть примерно так:

types/types.d.ts
~~~
export type SumArgs = {
  firstArgument: number;
  secondArgument: number;
};
~~~

App.tsx
~~~
import { SumArgs } from "./types/types";
~~~
