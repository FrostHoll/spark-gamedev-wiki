# Примеры хороших практик

В этой статье я приведу примеры некоторых практик из своих проектов. Эта страничка будет пополняться время от времени.

## Изменение поведения налету

В проекте визуальной новеллы у меня могут двигаться персонажи по экрану. Анимация движения задается из отдельного файла написанного на Lua, поэтому мне нужно в зависимости от указанного названия анимации по-разному двигать персонажа.

Первое, что приходит на ум, естественно, в методе движения персонажа сделать switch по указанной анимации, и в соответствии с ней менять уравнение интерполяции движения. 

Типа такого:

```csharp
void MoveCharacter(... string animation)
{
	// ...
	while (character.position != targetPosition)
	{
		switch (animation)
		{
			case "smooth":
				character.position = character.position + targetPosition * speed * Time.deltaTime;
				break;
			//... 
		}
	}
}
```

И этот switch должен прогоняться на каждом кадре. При добавлении нового уравнения, приходится переписывать код движения. **Звучит отвратно.**

Здесь отлично зайдет применение паттерна [Стратегия](https://metanit.com/sharp/patterns/3.1.php). Давайте сначала посмотрим на реальный код самого передвижения:

```csharp
public IEnumerator MoveCharacter(... MoveCharacterArgs args, ...)
{
    //...
    float elapsedTime = 0f;
    float duration = args.Duration;
    while (elapsedTime < duration)
    {
        float t = elapsedTime / duration;
        float interpolatedT = args.Animation.Evaluate(t); // <--
        target.localPosition = Vector3.Lerp(startPos, targetPos, interpolatedT);
        elapsedTime += Time.deltaTime;
        yield return null;
    }
    //...
}
```

Я убрал лишний код, который не касается текущей темы. Суть кода в том, что изменение анимации достигается за счет интерполяции времени движения. Проще говоря, я задаю правило, как двигается "ползунок" анимации, как если бы эта анимация была на таймлайне.

Заметьте, что время интерполируется из некой анимации из аргументов (строчка помеченная стрелочкой). Давайте взглянем на эти аргументы.

```csharp
public class MoveCharacterArgs
{
    public float? ToX { get; set; }

    public float? ToY { get; set; }

    public IInterpolationFunc Animation {  get; set; } // <--

    public float Duration { get; set; }

    public MoveCharacterArgs()
    {
        ToX = null;
        ToY = null;
        Animation = new LinearInterpolation();
        Duration = 1f;
    }
}
```

В анимация здесь - интерфейс `IInterpolationFunc`, который по умолчанию задается как линейная (в конструкторе). Эти аргументы формируются при компиляции команды, которую мы посмотрим позже. Давайте взглянем на интерфейс и пару реализаций.

```csharp
public interface IInterpolationFunc
{
    float Evaluate(float dt);
}
```

Вот сам интерфейс. Он определяет только один метод `Evaluate` (его использование мы видели в коде передвижения), который и должен интерполировать время. Сказать больше о нем нечего.

```csharp
public class LinearInterpolation : IInterpolationFunc
{
    public float Evaluate(float dt)
    {
        return dt;
    }
}

public class QuadraticInInterpolation : IInterpolationFunc
{
    public float Evaluate(float dt)
    {
        return dt * dt;
    }
}
```

И тут два примера классов, которые его реализуют. У линейной интерполяции (первый класс), как и должно быть, время никак не интерполируется. Результатом такого будет просто обычное перемещение персонажа из точки А в точку Б.

А вот у квадратичной (второй класс) - время возводится в квадрат. Если мы вспомним график параболы x<sup>2</sup> при x>0, то график сначала идет медленно, но дальше быстро разгоняется. Именно так и будет выглядеть движение персонажа.

И таким образом мы можем писать сколько угодно классов, которые реализуют свои уравнения интерполяций. Если заглянете в интернет, найдете там просто ужасающее количество таких уравнений. Давайте все-таки вернемся, и посмотрим как мы их используем.

Как я уже говорил, сама анимация выбирается при компиляции команды движения. Давайте взглянем на нее.

```csharp
public class MoveCharacterCommand : ScenarioCommand
{
    //...
    private MoveCharacterArgs _args;

    public MoveCharacterCommand(...)
    {
        //...
        if (moveInfo.TryGetString("animation", out var animation))
        {
            _args.Animation = animation switch
            {
                "linear" => new LinearInterpolation(),
                "quadIn" => new QuadraticInInterpolation(),
                "quadOut" => new QuadraticOutInterpolation(),
                _ => throw new ArgumentException("Invalid move animation argument", animation),
            };
        }
        //...
    }

    public override IEnumerator Execute()
    {
        return Context.CharacterManager.MoveCharacter(_charCode, _args, this);
    }
}
```

Итак, при создании команды (конструктор), мы берем из `moveInfo`, что является хранилищем параметров, полученный из Lua, строчку анимации. Тут уже по switch'у в аргументы `MoveCharacterArgs` в качестве анимации засовываем нужный класс, содержащий интерполяцию. И когда команда выполняется, она вызывает тот самый метод `MoveCharacter`, с которого мы начали. Весь код закольцевался.

Если вам интересно, вот как вызывается это все из Lua:

```lua
MoveCharacter("lilly", {x=0.3, animation="quadOut", duration=0.2})
```

**Итог**

Интерфейсы и в частности паттерн Стратегия позволяют "налету" изменять некоторые методы у классов по необходимости. Еще один пример - если у игрока есть какой-то предмет, то пули при попадании по врагу должны взрываться. Мы можем у пули добавить поле, которое определяет, что будет происходить, когда она столкнется. (как у меня поле `Animation` в `MoveCharacterArgs`). И при выстреле задавать это поведение, если у нас есть такой предмет. Реализовать это поведение через интерфейс (как `IInterpolationFunc`) и сделать реализации. Не только взрыва, но и телепортации, лечения, чего угодно. При этом при добавлении нового поведения у нас не меняются классы пули, выстрела, игрока.