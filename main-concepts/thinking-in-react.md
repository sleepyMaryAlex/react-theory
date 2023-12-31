# ?Thinking in React

Пример создания проекта:

1. __Разобьём интерфейс на составляющие.__

Первое, что нужно сделать — выделить отдельные компоненты (и подкомпоненты) в макете и дать им имена. Если вы работаете с дизайнерами, вполне возможно, что они уже как-то называют компоненты — вам стоит пообщаться! Например, слои Photoshop часто подсказывают имена для React-компонентов.

Можно применить принцип единой ответственности: каждый компонент должен заниматься какой-то одной задачей. Если функциональность компонента увеличивается с течением времени, его следует разбить на более мелкие подкомпоненты. Теперь, когда мы определили компоненты в нашем макете, давайте расположим их согласно иерархии.

2. __Создадим статическое приложение в React.__

Теперь, когда все компоненты расположены в иерархическом порядке, пришло время воплотить в жизнь наше приложение.

Самый лёгкий способ — создать версию, которая использует модель данных и рендерит интерфейс, но не предполагает никакой интерактивности. Разделять эти процессы полезно.

Написание статического приложения требует много печатания и совсем немного мышления. С другой стороны, создание интерактивного приложения подразумевает более глубокий мыслительный процесс и лишь долю рутинной печати.

Написание кода можно начать как сверху вниз, так и снизу вверх. Более простые приложения удобнее начать с компонентов, находящихся выше по иерархии. В более сложных приложениях удобнее в первую очередь создавать и тестировать подкомпоненты.

Компонент выше по иерархии будет передавать модель данных через пропсы. Благодаря одностороннему потоку данных, код работает быстро, но остаётся понятным.

3. __Определим минимальное (но полноценное) отображение состояния интерфейса.__

Чтобы сделать наш UI интерактивным, нужно, чтобы модель данных могла меняться со временем. В React это возможно с помощью состояния. Определите минимально необходимое состояние, которое нужно вашему приложению, всё остальное вычисляйте при необходимости.

Давайте рассмотрим данные по частям и определим, что должно храниться в состоянии:

* Передаются ли они от родителя через пропсы? Если так, тогда эти данные не должны храниться в состоянии компонента.
* Остаются ли они неизменными со временем? Если так, тогда их тоже не следует хранить в состоянии.
* Можете ли вы вычислить их на основании любых других данных в своём компоненте или пропсов? Если так, тогда это тоже не состояние.

4. __Определим, где должно находиться наше состояние.__

Итак, мы определили минимальный набор состояний приложения. Далее нам нужно выяснить, какой из компонентов владеет состоянием или изменяет его.

Где должно находиться состояние:

* Определите компоненты, которые рендерят что-то исходя из этого состояния.
* Найдите общий главенствующий компонент (компонент, расположенный над другими компонентами, которым нужно это состояние).
* Либо общий главенствующий компонент, либо любой компонент, стоящий выше по иерархии, должен содержать состояние.
* Если вам не удаётся найти подходящий компонент, то создайте новый исключительно для хранения состояния и разместите его выше в иерархии над общим главенствующим компонентом.

5. __Добавим обратный поток данных.__

Пока что наше приложение рендерится в зависимости от пропсов и состояния, передающихся вниз по иерархии. Теперь мы обеспечим поток данных в обратную сторону: наша задача сделать так, чтобы компоненты формы в самом низу иерархии обновляли состояние в родительских компонентах.
