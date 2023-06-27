# ?Lifting state up

Часто несколько компонентов должны отражать одни и те же изменяющиеся данные. Рекомендуется поднимать общее состояние до ближайшего общего предка.

Calculator - родительский компонент.

~~~
class Calculator extends React.Component<object, {temperature: string, scale: 'c' | 'f'}> {
  constructor(props: object) {
    super(props);
    this.state = {
      temperature: '',
      scale: 'c',
    }
  }

  tryConvert(temperature: string, convert: (input: number) => number) {
    const input = parseFloat(temperature);
    if (Number.isNaN(input)) {
      return '';
    }
    const output = convert(input);
    const rounded = Math.round(output * 1000) / 1000;
    return rounded.toString();
  }

  toCelsius(fahrenheit: number) {
    return (fahrenheit - 32) * 5 / 9;
  }

  toFahrenheit(celsius: number) {
    return (celsius * 9 / 5) + 32;
  }

  handleCelsiusChange(temperature: string) {
    this.setState({scale: 'c', temperature});
  }

  handleFahrenheitChange(temperature: string) {
    this.setState({scale: 'f', temperature});
  }

  render() {
    const { scale, temperature } = this.state;
    const celsius = scale === 'f' ? this.tryConvert(temperature, this.toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? this.tryConvert(temperature, this.toFahrenheit) : temperature;
    return (
      <div>
        <TemperatureInput scale="c" temperature={celsius} onTemperatureChange={this.handleCelsiusChange.bind(this)}/>
        <TemperatureInput scale="f" temperature={fahrenheit} onTemperatureChange={this.handleFahrenheitChange.bind(this)}/>
      </div>
    );
  }
}
~~~

TemperatureInput - дочерний компонент.

~~~
const scaleNames = {
  c: 'Цельсия',
  f: 'Фаренгейта'
};

class TemperatureInput extends React.Component<{scale: 'c' | 'f', temperature: string, onTemperatureChange: (temperature: string) => void}> {
  constructor(props: {scale: 'c' | 'f', temperature: string, onTemperatureChange: (temperature: string) => void}) {
    super(props);
  }

  handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    return (
      <fieldset>
        <legend>Введите температуру в градусах {scaleNames[this.props.scale]}:</legend>
        <input value={this.props.temperature}
              onChange={this.handleChange.bind(this)} />
      </fieldset>
    );
  }
}
~~~

Давайте посмотрим, что происходит:

* React вызывает функцию, указанную в `onChange` на DOM-элементе `<input>`. В нашем случае это метод `handleChange()` компонента `TemperatureInput`.
* Метод `handleChange()` в компоненте `TemperatureInput` вызывает `this.props.onTemperatureChange()` с новым требуемым значением. Его пропсы, включая `onTemperatureChange`, были предоставлены его родительским компонентом — `Calculator`.
* Когда `Calculator` рендерился ранее, он указал, что `onTemperatureChange` в компоненте `TemperatureInput` по шкале Цельсия — это метод `handleCelsiusChange` в компоненте `Calculator`, а `onTemperatureChange` компонента `TemperatureInput` по шкале Фаренгейта — это метод `handleFahrenheitChange` в компоненте `Calculator`. Поэтому один из этих двух методов `Calculator` вызывается в зависимости от того, какое поле ввода редактируется.
* Внутри этих методов компонент `Calculator` указывает React сделать повторный рендер себя, используя вызов `this.setState()` со значением нового поля ввода и текущей шкалой.
* React вызывает метод `render()` компонента `Calculator`, чтобы узнать, как должен выглядеть UI. Значения обоих полей ввода пересчитываются исходя из текущей температуры и шкалы. В этом методе выполняется конвертация температуры.
* React вызывает методы `render()` конкретных компонентов `TemperatureInput` с их новыми пропсами, переданными компонентом `Calculator`. Он узнает, как должен выглядеть UI.
* React DOM обновляет DOM, чтобы привести его в соответствие с нужными нам значениями в полях ввода. Отредактированное нами только что поле ввода получает его текущее значение, а другое поле ввода обновляется конвертированным значением температуры.

Для любых изменяемых данных в React-приложении должен быть один «источник истины». Обычно состояние сначала добавляется к компоненту, которому оно требуется для рендера. Затем, если другие компоненты также нуждаются в нём, вы можете поднять его до ближайшего общего предка.

Если что-то может быть вычислено из пропсов или из состояния, то скорее всего оно не должно находиться в состоянии. Например, вместо сохранения `celsiusValue` и `fahrenheitValue`, мы сохраняем только последнюю введённую температуру (`temperature`) и её шкалу (`scale`). Значение другого поля ввода можно всегда вычислить из них в методе `render()`.
