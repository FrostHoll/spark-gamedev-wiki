# Как работать с Git

Вообще я, конечно, скинул в сводке ссылки на статьи по Git, но, очень сомневаюсь, что вы их читали.

Итак, давайте разберемся, что такое Git, зачем он нам нужен, как им пользоваться, и причем тут GitHub.

## Введение

Разработчики в давние времена передавали друг другу файлы проекта либо через локальную сеть, либо по дискетам/дискам. В наши времена это сильно упростилось: коллеги заливают файлы на сервер, в частности через систему Git.

**Git** - система контроля версий. То есть каждый из ребят *заливает свою версию* проекта, при этом каждый *работает с последней версией*.

**GitHub** - это сервис-обертка для системы Git, которая предоставляет удобную работу со всем этим для полных нубиков.

Сами версии распределены по веткам. Главная ветка **main** должна быть чистенькой, то есть не содержать багов и т.д., чтобы любой мог скачать последнюю версию этой ветки, и *спокойно*, без проблем делать свою задачу.

Ветки - это, так скажем, параллельные истории версий проекта, которые в конечном итоге должны удачно слиться в главную. А каждая версия называется коммитом (commit).

## Общее описание работы

Когда мы работаем над проектом, мы создаем свою ветку и начинаем заниматься несчастным тыканьем по клавиатуре. Заметьте - над **вашей** задачей, никто кроме **вас** не работает. А значит, пока вы не доделаете свою задачу, ни у кого с вашим бедным кодом проблем не будет. Это крайне удобно, чтобы вы не зависели от завершения задач других ребят, и вам никто не мешал заниматься своей задачей (говнокодингом).

Для каждой ветки можно сделать **Fetch**, **Pull** и **Push**. Fetch - проверка появилась ли новая версия. Pull - вытащить последнюю версию, обновить все файлы, чтобы все было актуально. Push - закинуть свою версию. Важно знать, что **нельзя закинуть свою версию, если у вас не актуальные файлы**. Вам нужно сначала вытащить последнюю версию, затем уже закидывать свою. Все таки, версии в пределах одной ветки линейные.

Другой вопрос - а что если вы с кем-то работали над одним и тем же файлом? При попытке скачать последнюю версию случится самое страшное... *схлопывание вселенной*... *тепловая смерть*... короче - conflict. Что с этим делать? Либо уничтожать свои изменения, либо громко и горько плакать... Если такое случится, разберемся на месте.

Но до конфликтов внутри ветки, надеюсь, не дойдем. Помним? **Каждый создает свою ветку для каждой задачи**.

Так вот, вернемся к нашим баранам. Вы, как прилежный разработчик, считаете (именно считаете), что завершили свою задачу. И теперь ваши горе-результаты можно скинуть в основную ветку - к остальным ребятам. Для таких финтов с ушами делаем **Pull Request**. То есть, вы создаете заявку на внесение ваших изменений в main. На мой стол после этого попадает ваша ветка, и мне нужно дать согласие на слияние с главной. И вот тут начинается кое-что страшнее - Merge Conflict, когда ваша ветка содержит неактуальные изменения относительно main. Страшная вещь. Разрешать такие конфликты - целое искусство.

В целом, это все, что вам нужно знать. С остальным вы столкнетесь на практике. Вообще Git - штука удобная, особенно в руках, растущих из правильного места. Поэтому постигаем новые вершины путем беспощадного разбития собственной головушки об стену!

Можете, кстати, заглянуть на мой [профиль](https://github.com/FrostHoll) в GitHub, в качестве интереса.

## Рекомендации

### Написание коммитов

Когда вы создаете коммит, вы должны выбрать те файлы, в которые вы внесли изменения, чтобы они оказались в версии. После этого, вы должны дать **осмысленное** название коммита и его **описание**. В идеале, коммиты должны быть на английском. Но в качестве пробы разрешаю писать и на русском.

Приведу примеры своих коммитов (как надо писать) и один коммит моего бедного коллеги (как не надо).

> **pre-0.3**
> 
> -we finally can throw a ball
> 
> -inventory system is now fully replaced with pockets 
> 
> -added final stages of first scenario 
> 
> -added required voice lines 
> 
> -added colliders to almost every furniture 
> 
> -ui interactions now made with side button, not trigger 
> 
> -information windows fully redesigned 
> 
> -removed smoke in corridor (it's still not working)

&nbsp;

> **Level design**
> 
> -Добавил библиотеку vfx graph для более качественного дыма (надо проверить как работает в очках)
> 
>-слегка изменена основная комната( колайдеры переместил, зону телепортации расширил) 
>
>-небольшие изменения в коридоре (проверить освещенность) 
>
>-улица убогая как всегда, добавил новые модели 
>
>-скомпилировать не получилось. хз что за ошибка

И как **не** надо:

>**1**
>
>я еблуша забыл ctrl+s нажать

### Написание Pull Request

В этом случае применяется все то, что написано выше, только название и описание нужно делать к **целой** ветке, а не к одиночным изменениям.