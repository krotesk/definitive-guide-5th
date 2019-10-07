---
description: Погружение в диалплан
---

# Глава 10

> _Для получения списка всех способов, которыми технология не смогла улучшить качество жизни, нажмите три._
>
> -- Элис Кан

Хорошо. Основы диалплана позади, но вы знаете что это еще не все. Если вы еще не разобрались с [Главой 6](glava-06.md), пожалуйста, вернитесь и прочтите ее еще раз. Мы собираемся перейти к более сложным темам.

## Выражения и манипуляции с переменными

Мы начинаем наше погружение в более глубокие аспекты диалпланов, пришло время познакомить вас с несколькими инструментами, которые значительно увеличат мощь, которую вы можете использовать в своем диалплане. Эти конструкции добавляют невероятный интеллект к вашему диалплану, позволяя ему принимать решения на основе различных критериев, которые вы определяете. Надень свой мыслительный колпачок, и давай начнем.

{% hint style="info" %}
**Примечание**

В этой главе мы используем лучшие практики, которые были разработаны на протяжении многих лет при создании диалплана. Основное, что вы заметите, это то, что все первые приоритеты начинаются с приложения `NoOp()` \(что просто означает No Operation - отсутствие операции; ничего функционального не произойдет\). Другое заключается в том, что все следующие строки будут начинаться с `same => n`что является ярлыком, который говорит: “используйте то же \(same\) расширение, что и ранее."К роме того, отступ - это два пробела.
{% endhint %}

### Basic Expressions

Выражения - это комбинации переменных, операторов и значений, которые строятся вместе для получения результата. Выражение может проверять значения, изменять строки или выполнять математические вычисления. Допустим, у нас есть переменная под названием `COUNT`. На простом английском языке два выражения, использующие эту переменную, могут быть \[`COUNT` плюс 1\] или \[`COUNT` делить на 2\]. Каждое из этих выражений имеет определенный результат или значение, зависящее от значения данной переменной.

В Asterisk выражения всегда начинаются со знака доллара и открывающей квадратной скобки и заканчиваются закрывающей квадратной скобкой, как показано здесь:

```text
$[expression]
```

Таким образом, мы запишем наши два примера следующим образом:

```text
$[${COUNT} + 1]
$[${COUNT} / 2]
```

Когда Asterisk встречает выражение в диалплане, он заменяет все выражение результирующим значением. Важно отметить, что это происходит _после_ подстановки переменных. Для демонстрации рассмотрим следующий код: [1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html#asterisk-CHP-6-FN-1)

```text
exten => 321,1,NoOp()
 same => n,Answer()
 same => n,Set(COUNT=3)
 same => n,Set(NEWCOUNT=$[${COUNT} + 1])
 same => n,SayNumber(${NEWCOUNT})
```

Во втором приоритете мы присваиваем значение `3` переменной с именем `COUNT`.

В третьем приоритете задействовано только одно приложение - `Set()`, но на самом деле происходят три вещи:

1. Asterisk заменяет `${COUNT}` на число `3` в выражении. Выражение эффективно становится таким: `same => n,Set(NEWCOUNT=$[3 + 1])`
2. Asterisk вычисляет выражение, прибавляя `1` к `3`, и заменяет его вычисленным значением `4`: `same => n,Set(NEWCOUNT=4)`
3. Приложение `Set()` присваивает значение `4` новой переменной `COUNT`.

Третий приоритет просто вызывает приложение `SayNumber()`, которое проговаривает текущее значение переменной `${NEWCOUNT}` \(устанавливается в значение `4` во втором приоритете\).

Попробуйте это в своём диалплане.

### Операторы

Когда вы создаете диалплан Asterisk, вы действительно пишете код на специализированном языке сценариев. Это означает, что диалплан Asterisk, как и любой язык программирования, распознает символы, называемые операторами, которые позволяют управлять переменными. Давайте рассмотрим типы операторов, которые доступны в Asterisk:

_Логические операторы_

Эти операторы оценивают "истинность" утверждения. В вычислительных терминах это по существу относится к тому, является ли утверждение чем-то или ничем \(ненулевым или нулевым, истинным или ложным, включенным или выключенным и т.д.\). Логическими операторами являются:

_`expr1 | expr2`_

Этот оператор \(называемый оператором “или” или “пайп”\) возвращает оценку _`expr1`_ если она истинна \(ни одна строка не равна нулю\). В противном случае он возвращает оценку _`expr2`_.

_`expr1 & expr2`_

Этот оператор \(называемый “и”\) возвращает вычисление _`expr1`_, если оба выражения истинны \(т.е. ни одно из выражений не является в пустой строкой или нулем\). В противном случае возвращает ноль.

_`expr1 {=, >, >=, <, <=, !=} expr2`_

Эти операторы возвращают результаты сравнения целых чисел, если оба аргумента являются целыми числами; в противном случае возвращают результаты сравнения строк. Результат каждого сравнения равен `1`, если указанное отношение истинно, или `0` если отношение ложно. \(Если вы выполняете сравнение строк, они будут выполняться в соответствии с текущими локальными настройками вашей операционной системы.\)

_Математические операторы_

Хотите выполнить расчет? Вам понадобится один из них:

_`expr1 {+, -} expr2`_

Эти операторы возвращают результат сложения или вычитания целочисленных аргументов.

_`expr1 {*, /, %} expr2`_

Возвращают, соответственно, результат умножения, целочисленного деления или остатка деления целочисленных аргументов.

_Операторы регулярных выражений_

Вы также можете использовать операторы регулярных выражений в Asterisk:

{% hint style="info" %}
**Примечание**

Дополнительную информацию об особенностях работы оператора регулярного выражения в Asterisk можно найти на [сайте Уолтера Докса](https://wjd.nu/notes/2011#asterisk-dialplan-peculiarities-regex).
{% endhint %}

_`expr1 : expr2`_

Этот оператор сопоставляет _`expr1`_ с _`expr2`_, где _`expr2`_ должно быть регулярным выражением.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html#asterisk-CHP-6-FN-2) Регулярное выражение привязывается к началу строки с неявным `^`.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html#asterisk-CHP-6-FN-3)

Если шаблон не содержит подвыражения, возвращается количество совпадающих символов. Это вернет `0`, если совпадений не найдено. Если шаблон содержит подвыражение -- \\(... \\) -- возвращается строка, соответствующая `\1`. Если совпадение не найдено, возвращается пустая строка.

_`expr1 =~ expr2`_

Этот оператор работает так же, как и оператор `:`, за исключением того, что он не привязан к началу.

## Функции диалплана

Функции диалплана позволяют добавить больше мощи к вашим выражениям; вы можете думать о них как об интеллектуальных переменных. Функции диалплана позволяют вычислять длины строк, даты и время, контрольные суммы MD5 и т.д. в пределах выражений диалплана.

{% hint style="info" %}
**Примечание**

Вы увидите использование функции `Playback(silence/1)` во всех примерах в этой главе. Мы делаем так поскольку она ответит на линию, если еще не ответили и воспроизведёт некоторую тишину на линии. Это позволяет другим приложениям, таким как `SayNumber()`, воспроизводить звук без пропусков.
{% endhint %}

### Синтаксис

Функции диалплана имеют следующий базовый синтаксис:

```text
FUNCTION_NAME(argument)
```

Вы ссылаетесь на имя функции так же, как и на имя переменной, но на значение функции ссылаются с добавлением знака доллара, открывающейся и закрывающейся фигурной скобки:

```text
${FUNCTION_NAME(argument)}
```

Функции также могут инкапсулировать другие функции, например:

```text
${FUNCTION_NAME(${FUNCTION_NAME(argument)})}
 ^             ^ ^             ^        ^^^^
 1             2 3             4        4321
```

Как вы, вероятно, уже поняли необходимо быть очень осторожными, чтобы убедиться в наличии соответствующих круглых и фигурных скобок. В предыдущем примере мы обозначили открывающие круглые и фигурные скобки цифрами, а их соответствующие закрывающие аналоги - теми же цифрами.

### Примеры функций диалплана

Функции часто используются совместно с приложением `Set()` для получения или установки значения переменной. В качестве простого примера рассмотрим функцию `LEN()`. Эта функция вычисляет длину строки своего аргумента:

```text
exten => 205,1,Answer()
 same => n,SayDigits(123)
 same => n,SayNumber(123)
 same => n,SayNumber(${LEN(123)})
```

Давайте рассмотрим еще один простой пример. Если бы мы хотели установить один из различных таймаутов канала, мы могли бы использовать функцию `TIMEOUT()`. Функция `TIMEOUT()` принимает три аргумента: `absolute`, `digit` и `response`. Чтобы установить тайм-аут с помощью функции `TIMEOUT()`, мы могли бы использовать приложение `Set()`, например:

```text
exten => 206,1,Answer()
 same => n,Set(TIMEOUT(response)=1)
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten()  ; TIMEOUT() установлен в значение 1
 same => n,Playback(like_to_tell_valid_ext)
 same => n,Set(TIMEOUT(response)=5)
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten()  ; Теперь должно быть 5 секунд
 same => n,Playback(укажите_действительный_файл)
 same => n,Hangup()
```

Обратите внимание на отсутствие `${ }` вокруг назначения с помощью функции. Так же, как если бы мы присваивали значение переменной, мы присваиваем значение функции без использования инкапсуляции `${}`; однако, если мы хотим использовать значение, возвращаемое функцией, то нам нужна инкапсуляция.

```text
exten => 207,1,Answer()
 same => n,Set(TIMEOUT(response)=1)
 same => n,SayNumber(${TIMEOUT(response)})
 same => n,Set(TIMEOUT(response)=5)
 same => n,SayNumber(${TIMEOUT(response)})
 same => n,Hangup()
```

Вы можете получить список всех активных функций с помощью следующей команды CLI:

```text
*CLI> core show functions
```

Или по определенной функции, например `CALLERID()`, команда:

```text
*CLI> core show function CALLERID
```

Ближе к концу этой главы мы рассмотрим несколько функций, с которыми вы захотите поэкспериментировать. Далее в книге мы покажем вам, как создавать функции на основе баз данных с помощью `func_odbc`.

## Условное ветвление

Расширенная логика, предоставляемая через выражения и функции, позволит вашему диалплану принимать более сложные решения, что часто приводит к _условному ветвлению_.

### Приложение GotoIf\(\)

Ключом к условному ветвлению является приложение `GotoIf()`. `GotoIf()` вычисляет выражение и отправляет вызывающий объект в определенное место назначения в зависимости от того, имеет ли выражение значение истинности или лжи.

`GotoIf()` использует специальный синтаксис, часто называемый _условным синтаксисом_:

```text
GotoIf(expression?destination1:destination2)
```

Если выражение принимает значение "истина", вызывающий объект отправляется в _`destination1`_. Если выражение принимает значение "ложь", вызывающий объект отправляется в _`destination2`_. Итак, что же такое истина и что такое ложь? Пустая строка и число `0` оцениваются как ложь. Все остальное оценивается как истина.

Каждый из пунктов назначения может быть одним из следующих:

* Метка приоритета в пределах одного расширения, например `weasels`
* Расширение и метка приоритета в одном контексте, например `123,weasels`
* Контекст, расширение и метка приоритета, такие как `incoming,123,weasels`

Давайте используем `GotoIf()` в качестве примера. Вот небольшое приложение для подбрасывания монет. Вызовите его несколько раз, чтобы проверить правильность.

```text
exten => 209,1,Noop(Test use of conditional branching to labels)
 same => n,GotoIf($[ ${RAND(0,1)} = 1 ]?weasels:iguanas)
; same => n,GotoIf(${RAND(0,1)}?weasels:iguanas) ;тоже работает, но не в каждой ситуации
 same => n(weasels),Playback(weasels-eaten-phonesys) ; ПРИМЕЧАНИЕ: ТО ЖЕ РАСШИРЕНИЕ
 same => n,Hangup()
 same => n(iguanas),Playback(office-iguanas) ; ВСЕ ТО ЖЕ РАСШИРЕНИЕ
 same => n,Hangup()
```

{% hint style="info" %}
**Примечание**

Вы заметите, что мы использовали приложение `Hangup()` после каждого использования приложения `Playback()`. Это делается для того, чтобы при переходе к метке `weasels` вызов останавливался до того, как он попадет на звуковой файл _office-iguanas_. Становится все более распространенным видеть расширения, разбитые на несколько компонентов \(защищенных друг от друга командой `Hangup()`\), каждый из которых представляет собой отдельную последовательность шагов, выполняемых после `GotoIf()`.
{% endhint %}

{% hint style="info" %}
#### Предоставление только ложного условного пути

Любой из пунктов назначения может быть опущен \(но не оба\). Если выражение оценивается как пустое назначение, Asterisk просто переходит к следующему приоритету в текущем расширении.

Мы могли бы выполнить предыдущий пример следующим образом:

```text
exten => 209,1,Noop(Test use of conditional branching)
 same => n,GotoIf($[ ${RAND(0,1)} = 1 ]?:iguanas)
 same => n,Playback(weasels-eaten-phonesys) ; больше нет ярлыка weasels
 same => n,Hangup()
 same => n(iguanas),Playback(office-iguanas) ; ОБРАТИТЕ ВНИМАНИЕ ЧТО ЭТО ТО ЖЕ РАСШИРЕНИЕ
 same => n,Hangup()
```

Между `?` и `:` ничего нет, так что если оператор оценивается как истина, выполнение будет продолжено на следующем шаге. Поскольку это то, что мы хотим, ярлык не нужен.

Мы действительно не рекомендуем делать так, потому что это трудно читать. Тем не менее, вы увидите такие диалпланы, поэтому хорошо знать, что этот синтаксис технически корректен.
{% endhint %}

Вместо того, чтобы использовать метки \(лейблы\), мы могли бы также отправить вызов на различные расширения. Поскольку они недоступны, мы можем использовать буквы, а не цифры для расширения “номера". В этом примере условная ветвь отправляет вызов на совершенно разные расширения в одном и том же контексте. В остальном результат тот же.

```text
exten => 210,1,Noop(Test use of conditional branching to extensions)
 same => n,GotoIf($[ ${RAND(0,1)} = 1 ]?weasels,1:iguanas,1)
exten => weasels,1,Playback(weasels-eaten-phonesys) ; РАЗЛИЧНЫЕ РАСШИРЕНИЯ
 same => n,Hangup()
exten => iguanas,1,Playback(office-iguanas) ; ТАКЖЕ РАЗЛИЧНЫЕ РАСШИРЕНИЯ
 same => n,Hangup()
```

Рассмотрим еще один пример условного ветвления. На этот раз мы будем использовать оба `Goto()` и `GotoIf()` для обратного отсчета от `5`, а затем повесим трубку:

```text
exten => 211,1,NoOp()
 same => n,Answer()
 same => n,Set(COUNT=5)
 same => n(start),GotoIf($[ ${COUNT} > 0 ]?:goodbye)
 same => n,SayNumber(${COUNT})
 same => n,Set(COUNT=$[ ${COUNT} - 1 ])
 same => n,Goto(start)
 same => n(goodbye),Playback(vm-goodbye)
 same => n,Hangup()
```

Давайте проанализируем этот пример. Во втором приоритете мы задаем переменную `COUNT` равную `5`. Далее, проверяем, чтобы увидеть если `COUNT` больше `0`. Если это так, мы переходим к следующему приоритету. \(Не забывайте, что если мы опустим назначение в приложении `GotoIf()`, управление перейдет к следующему приоритету.\) Оттуда мы произносим число, вычитаем `1` из числа и возвращаемся к метке приоритета `start`. Опять же, если `COUNT` меньше или равен `0`, управление переходит к метке приоритета `goodbye`; в противном случае мы запускаем цикл еще раз.

{% hint style="info" %}
#### Кавычки и префиксы переменных в условных ветвлениях

Сейчас самое время воспользоваться моментом, чтобы посмотреть на некоторые небрежные вещи с условными ветвями. В Asterisk недопустимо иметь нулевое значение по обе стороны от оператора сравнения. Давайте рассмотрим примеры, которые могли бы привести к ошибке:

```text
$[ = 0 ]
$[ foo = ]
$[ > 0 ]
$[ 1 + ]
```

Любой из наших примеров вызовет такое предупреждение:

```text
WARNING[28400][C-000000eb]: ast_expr2.fl:470 ast_yyerror: ast_yyerror():
syntax error: syntax error, unexpected '=', expecting $end; Input:
 = 0
 ^
```

Это маловероятно \(если у вас нет опечатки\), что вы целенаправленно реализуете что-то из наших примеров. Однако, когда вы выполняете математическое действие или сравнение с неназначенной переменной канала, это фактически то, что вы делаете.

Примеры, которые мы использовали, чтобы показать вам как работает условное ветвление, не являются недопустимыми. Мы сначала инициализировали переменную и можем ясно видеть, что переменная канала, которую мы используем в нашем сравнении, была установлена, поэтому мы в безопасности. Но что, если вы не всегда так уверены?

В Asterisk строки необязательно должны быть заключены в двойные или одинарные кавычки, как во многих языках программирования. Фактически, если вы используете двойные или одинарные кавычки, это будет буквенной конструкцией в строке. Если мы посмотрим на следующие фрагменты расширения...

```text
same => n,Set(TEST_1=foo)
 same => n,Set(TEST_2='foo')
 same => n,NoOp(Are TEST_1 and TEST_2 equiv? $[${TEST_1} = ${TEST_2}])
```

...мы должны отметить, что значение, возвращаемое нашим сравнением в `NoOp()`, не будет значением `1` \(значения совпадают или истина\), возвращаемое значение будет 0 \(значения не совпадают или ложь\).

Мы можем использовать это в своих интересах при выполнении сравнений, обертывая наши переменные канала в одинарные или двойные кавычки. Делая это, мы удостоверяемся, что даже когда переменная канала не может быть установлена, наше сравнение будет допустимым синтаксисом.

В следующем примере мы получим ошибку:

```text
exten => 212,1,NoOp()
 same => n,GotoIf($[ ${TEST} != valid ]?error_handling)
 same => n,Hangup() ; We're getting an error and ending up here
 same => n(error_handling),Playback(goodbye)
 same => n,Hangup()
```

Однако, мы можем обойти это, обернув то, что мы сравниваем, в дополнительные символы \(в данном случае кавычки\). Тот же пример, но сделан допустимым:

```text
exten => 213,1,NoOp()
 same => n,GotoIf($[ "${TEST}" != "valid" ]?error_handling)
 same => n,Hangup()
 same => n(error_handling),Playback(goodbye)
 same => n,Hangup()
```

Даже если `${TEST}` не была установлена \(другими словами, она не существует и поэтому не имеет значения\), мы все равно делаем сравнение чего-то:

`$["" != "valid"]`

Если вы привыкнете распознавать эти ситуации и использовать методы обертки и префикса, которые мы описали, вы напишете гораздо более безопасные диалпланы.

Обратите внимание еще раз, что символ кавычки не имеет никакого особого значения здесь. Мы использовали его, потому что это логический символ для этой цели. Следующее тоже работает:

```text
same => n,GotoIf($[_${TEST}_ != _valid_]?error_handling)
;OR
 same => n,GotoIf($[AAAAA${TEST}AAAAA != AAAAAvalidAAAAA]?error_handling)
```

Не все символы будут работать, так как некоторые могут иметь другие значения для Asterisk и вызвать проблемы. Оставайтесь с кавычками и всё должно быть в порядке.
{% endhint %}

Классический пример условного ветвления ласково называют логикой "психо-экс". Если идентификационный номер абонента входящего вызова совпадает с номером телефона человека, с которым вы больше никогда не захотите разговаривать, Asterisk выдает другое сообщение, чем для любого другого абонента. Хотя он несколько прост и примитивен, это хороший пример для изучения условного ветвления в диалплане Asterisk.

В этом примере используется функция `CALLERID()`, которая позволяет получить информацию об идентификаторе вызывающего абонента при входящем вызове. Предположим, ради этого примера, что номер телефона жертвы 888-555-1212:4

```text
exten => 214,1,NoOp(CALLERID(num): ${CALLERID(num)} CALLERID(name): ${CALLERID(name)})
 same => n,GotoIf($[ ${CALLERID(num)} = 8885551212 ]?reject:allow)
 same => n(allow),Dial(${UserA_DeskPhone})
 same => n,Hangup()
 same => n(reject),Playback(abandon-all-hope)
 same => n,Hangup()
```

В приоритете `1` мы вызываем приложение `GotoIf()`. Он сообщает Asterisk перейти к приоритету метки `reject`, если номер идентификатора вызывающего абонента соответствует `8885551212` и в противном случае перейти к приоритету метки `allow` \(мы могли бы просто опустить имя метки в результате чего `GotoIf()` просто провалился\).[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html#idm46178406801064) Если CallerID абонента совпадает, управление вызовом переходит к метке приоритета `reject`, которая воспроизводит тонкий намёк нежелательному абоненту. В противном случае вызов пытается набрать получателя по каналу, на который ссылается глобальная переменная `UserA_DeskPhone`.

### Условное ветвление по времени с GotoIfTime\(\)

Другой способ использования условного ветвления в диалплане - это использование приложения `GotoIfTime()`. В то время как `GotoIf()` оценивает выражение для дальнейших действий, `GotoIfTime()` смотрит на текущее системное время и использует его, чтобы решить, следует ли следовать другой ветви в диалплане.

Наиболее очевидное использование этого приложения - это озвучить вашим абонентам другое приветствие до и после рабочих часов.

Синтаксис приложения `GotoIfTime()` выглядит следующим образом:

```text
GotoIfTime(times,days_of_week,days_of_month,months?label)
```

Короче говоря, `GotoIfTime()` отправляет вызов на указанный _`label`_, если текущая дата и время соответствуют критериям, указанным _`times, days_of_week, days_of_month`_ и _`months`_. Давайте рассмотрим каждый аргумент более подробно:

_`times`_

Это список одного или нескольких временных диапазонов в 24-часовом формате. Например, с 9:00 утра до 5:00 вечера будет указано как 09:00-17:00. День начинается в 0:00 и заканчивается в 23:59.

{% hint style="info" %}
**Примечание**

Стоит отметить, что время будет правильно оборачиваться. Таким образом, если вы хотите указать время закрытия вашего офиса, то можете указать 18:00-9:00 в параметре _`times`_, и оно будет работать как и ожидалось. Обратите внимание, что этот метод работает также и для других компонентов `GotoIfTime()`. Например, вы можете написать `sat-sun`, чтобы указать выходные дни.
{% endhint %}

_`days_of_week`_

Это список из одного или нескольких дней недели. Дни должны быть указаны как `mon`, `tue`, `wed`, `thu`, `fri`, `sat` и/или `sun`. С понедельника по пятницу будет выражаться как `mon-fri`. Вторник и четверг будут выражены как `tue&thu`.

{% hint style="info" %}
**Примечание**

Обратите внимание, что можно указать совокупность диапазонов и одного дня, как: `sun-mon&wed&fri-sat` или более просто: `wed&fri-mon`.
{% endhint %}

_`days_of_month`_

Это список чисел дней месяца. Дни указываются цифрами от `1` до `31`. С 7-го по 12-е число будет выражено как `7-12`, а 15-е и 30-е числа месяца будут записаны как `15&30`. Это может быть полезно для праздников, которые обычно приходятся на один и тот же день месяца, но не на один и тот же день недели.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html#idm46178406763720)

_`months`_

Это список из одного или нескольких месяцев в году. Месяцы должны быть записаны как `jan-apr` для диапазона и разделены амперсандами когда требуется включить не месяцы не последовательно как например `jan&mar&jun`. Вы также можете комбинировать их так: `jan-apr&jun&oct-dec`.

Если вы хотите сопоставить все возможные значения для любого из этих аргументов, просто поставьте `*` в этом аргументе.

Аргумент `label` может быть любым из следующих:

* Метка приоритета в пределах одного расширения, например `time_has_passed` 
* Расширение и приоритет в одном контексте, например `123, time_has_passed`
* Контекст, расширение, а также приоритет, например `incoming,123,time_has_passed` 

Теперь, когда мы рассмотрели синтаксис, давайте рассмотрим несколько примеров. Следующий пример будет соответствовать с _9:00 утра до 5:59 вечера_, с _понедельника по пятницу_, в _любой день месяца_, в _любом месяце года_:

```text
exten => s,1,NoOp()
 same => n,GotoIfTime(09:00-17:59,mon-fri,*,*?open,s,1)
```

Если вызывающий абонент звонит в течение этих часов, вызов будет направлен на первый приоритет расширения `start` в контексте с именем `open`. Если вызов выполняется вне указанного времени, он просто продолжит работу со следующего приоритета текущего расширения. Мы собираемся добавить новый контекст с именем `[closed]` сразу после примера соответствия шаблону `55512XX` и изменить контекст `[Test Menu]`, который мы создали в [Главе 6](glava-06.md), чтобы обработать наше новое правило по времени.

```text
exten => _55512XX,1,Answer()
 same => n,Playback(tt-monkeys)
 same => n,Hangup()
exten => *98,1,NoOp(Access voicemail retrieval.)
 same => n,VoiceMailMain()
[closed]
exten => start,1,Noop(after hours handler)
 same => n,Playback(go-away2)
 same => n,Hangup()
[TestMenu]
exten => start,1,Noop(main autoattendant)
 same => n,GotoIfTime(16:59-08:00,mon-fri,*,*?closed,start,1)
 same => n,GotoIfTime(11:59-09:00,sat,*,*?closed,start,1)
 same => n,GotoIfTime(00:00-23:59,sun,*,*?closed,start,1)
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Dial(${UserA_DeskPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
```

## GoSub

Приложение диалплана `GoSub()` позволяет отправить вызов в отдельный раздел диалплана, сделать что-то полезное, а затем вернуть вызов в точку в диалплане, откуда он пришел. Вы можете передать аргументы в `GoSub()`, а также получить от него код возврата. Оно немного увеличивает функциональность вашего диалплана.

{% hint style="info" %}
**Примечание**

Подпрограммы являются важнейшей способностью в любом языке программирования, и в не меньшей степени в диалплане Asterisk. Для тех, кто новичок в программировании: подпрограмма позволяет создать блок универсального кода, который может быть повторно использован различными частями диалплана, чтобы избежать повторения. Подумайте о ней как о шаблоне в текстовом документе или пустой форме, и у вас появится представление. Как только вы увидите их в действии, должно стать ясно, насколько полезными они могут быть.
{% endhint %}

### Определение подпрограмм

При использовании `GoSub()` в диалплане нет особых требований к именованию. Фактически, вы можете использовать `GoSub()` в том же контексте и расширении если пожелаете. В большинстве случаев, однако, ваши подпрограммы должны быть написаны в отдельных контекстах: один контекст для каждой подпрограммы. При создании контекста, мы рекомендуем добавить к имени  `sub`, чтобы знать что контекст вызывается из приложения `GoSub()`.

Давайте рассмотрим очевидный пример того, где подпрограмма была бы полезна.

Как вы могли заметить, при создании нашего примера диалплана для пользователей, которых мы добавили, логика диалплана для каждого пользователя может потребовать несколько строк кода.

```text
[sets]
exten => 100,1,Dial(${UserA_DeskPhone},12)
 same => n,Voicemail(100@default)
 same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?busy:unavail)
 same => n(unavail),VoiceMail(100@default,u)
 same => n,Hangup()
 same => n(busy),VoiceMail(100@default,b)
 same => n,Hangup()
exten => 101,1,Dial(${UserA_SoftPhone})
 same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?busy:unavail)
 same => n(unavail),VoiceMail(101@default,u)
 same => n,Hangup()
 same => n(busy),VoiceMail(101@default,b)
 same => n,Hangup()
exten => 102,1,Dial(${UserB_DeskPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 103,1,Dial(${UserB_SoftPhone})
 same => n,Hangup()
```

Мы предоставили только двум пользователям реальную, рабочую голосовую почту, и определили только четыре телефона как внутренние номера, и все же у нас уже есть беспорядок в виде повторяющегося кода, который будет все труднее поддерживать и расширять. Это быстро станет неуправляемым, если мы не найдем способа получше.

Давайте напишем подпрограмму для обработки набора номера наших пользователей. Добавьте следующее в самый конец вашего диалплана:

```text
; SUBROUTINES
[subDialUser]
exten => _[0-9].,1,Noop(Dial extension ${EXTEN},channel: ${ARG1}, mailbox: ${ARG2})
 same => n,Noop(mboxcontext: ${ARG3}, timeout ${ARG4})
 same => n,Dial(${ARG1},${ARG4})
 same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?busy:unavail)
 same => n(unavail),VoiceMail(${ARG2}@${ARG3},u)
 same => n,Hangup()
 same => n(busy),VoiceMail(${ARG2}@${ARG3},b)
 same => n,Hangup()
```

Теперь измените верхнюю часть своего диалплана следующим образом:

```text
[OLD_sets] ; что было [sets] теперь [OLD_sets] (называйте как угодно, имя изменить недолго)
exten => 100,1,Dial(${UserA_DeskPhone},12)
 same => n,Voicemail(100@default)
 same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?busy:unavail)
;(и тд)
```

Мы переименовали наш контекст `[sets]` , который, конечно, сломает наш диалплан, так как наши телефоны входят в диалплан в нем. Итак, мы собираемся снова добавить его немного ниже:

```text
exten => 103,1,Dial(${UserB_SoftPhone})
 same => n,Hangup()
[sets]
exten => 110,1,Dial(${UserA_DeskPhone}&${UserA_SoftPhone}&${UserB_SoftPhone})
 same => n,Hangup()
;(etc)
```

Итак, теперь у нас снова есть наш контекст `[sets]`, а также `[OLD_sets]`, в котором есть наш старый, осиротевший код. Как мы набираем наши телефоны? Как эта подпрограмма, которую мы только что написали, поможет нам?

```text
exten => 103,1,Dial(${UserB_SoftPhone})
 same => n,Hangup()
[sets]
;subDialUser args:
; - ARG1 канал(ы) для вызова
; - ARG2 почтовый ящик
; - ARG3 контекст почтового ящика
; - ARG4 Тайм-аут
exten => 100,1,Gosub(subDialUser,${EXTEN},1(${UserA_DeskPhone},${EXTEN},default,12))
exten => 101,1,Gosub(subDialUser,${EXTEN},1(${UserA_SoftPhone},${EXTEN},default,3))
exten => 102,1,Gosub(subDialUser,${EXTEN},1(${UserB_DeskPhone},${EXTEN},default,6))
exten => 103,1,Gosub(subDialUser,${EXTEN},1(${UserB_SoftPhone},${EXTEN},default,24))
exten => 110,1,Dial(${UserA_DeskPhone}&${UserA_SoftPhone}&${UserB_SoftPhone})
 same => n,Hangup()
```

Сохраните его, перезагрузите диалплан и выполните несколько тестовых вызовов. Поиграйте с параметрами и посмотрите что изменится. Добавьте несколько почтовых ящиков в свою базу данных и посмотрите, что произойдет. Если вы вдохновлены, напишите новую подпрограмму `subDialUserNEW` и посмотрите что сможете придумать. На этом этапе вы также можете удалить весь код в контексте `[OLD_sets]`, поскольку он теперь заброшен, но вы также можете оставить его там, поскольку он не причиняет вреда.

Теперь, вы можете добавить сотни расширений, и каждое из них будет использовать только одну строку диалплана.

Всякий раз, когда вы обнаружите, что где-то пишете дубликат кода диалплана, остановитесь. Вполне вероятно, что пришло время написать подпрограмму.

### Возврат из подпрограммы

Приложение диалплана `GoSub()` не возвращается автоматически после выполнения подпрограммы. Если вы закончили с вызовом, то можете, конечно, использовать `Hangup()`однако, если вы не хотите отключаться, а скорее вернуть вызов оттуда, откуда он пришел, вы можете использовать приложение `Return()`.

Поскольку вы можете вложить подпрограмму в подпрограмму, а также выполнять их одну за другой, когда попадаете в более сложные подпрограммы, то вскоре обнаружите, что это весьма полезная возможность.

## Локальные \(Local\) каналы

Local channels are a method of executing other areas of the dialplan from the Dial\(\) application \(as opposed to sending the call out a channel\). Think of them as subroutines you can call from within Dial\(\).

They may seem like a bit of a strange concept when you first start using them, but believe us when we tell you they can be the answer to a problem you can’t figure out any other way. You will almost certainly want to make use of them when you start writing advanced dialplans. The best way to illustrate the use of local channels is through an example. Let’s suppose we have a situation where we need to ring multiple people, but we need to provide delays of different lengths before dialing each of the members. The use of local channels is the solution to the problem.

With the Dial\(\) application, you can certainly ring multiple endpoints \(see extension 110 in your dialplan for an example of this\), but all three channels will ring at the same time, and for the same length of time.

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

 same =&gt; n,Hangup\(\)

However, let’s say we want to introduce some delays prior to ringing a user, and also stop ringing locations at different times. Using local channels gives us independent control over each of the channels we want to dial, so we can introduce delays and control the period of time for which each channel rings independently.

Let’s say we have a small company, where the receptionist is primarily responsible for the incoming calls, but there are also two team members who are tasked with backing up reception, and finally the owner wants to help out if need be too.

These are the requirements:

* The reception phone should ring right away, and keep ringing and not stop until answered.
* The team member phones shouldn’t ring for the first 9 seconds, at which point they can ring until answered.
* The owner’s phone should only ring if the call has gone on for 12 seconds with no answer. Also, we’re pretending it’s a cell phone, and thus should stop ringing 18 seconds later so that the call is not answered by the cell phone voicemail.

We’ll use our existing configured channels to play the various roles. If you have any way to do so, please try to have them all registered somewhere so they can all ring when called. It’ll give you a much better idea of what’s going on when testing.[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406696296)

This is a great time for a subroutine:

\[subDialDelay\]

exten =&gt; \_\[a-zA-Z0-9\].,1,Noop\(channel ${ARG1}, pre-delay ${ARG2}, timeout ${ARG3}\)

; same =&gt; n,Progress\(\) ; Optional; Signals back that the call is proceeding

{% hint style="info" %}

{% endhint %}

 same =&gt; n,Wait\(${ARG2}\) ; how long to wait before dialing

 same =&gt; n,Dial\(${ARG1},${ARG3}\) ; timeout can be blank \(infinite\)

 same =&gt; n,Hangup\(\)

**Note**

You already have a subroutine at the bottom of the file. Add this one down there too so all your subroutines end up grouped together.

Now we want a context in which we’ll build out the extensions to be used by the local channel:

;LOCAL CHANNELS

\[localDialDelay\]

exten =&gt; receptionist,1,Gosub\(subDialDelay,${EXTEN},1\(${UserA\_DeskPhone},0,600\)\)

exten =&gt; team\_one,1,Gosub\(subDialDelay,${EXTEN},1\(${UserA\_SoftPhone},9,600\)\)

exten =&gt; team\_two,1,Gosub\(subDialDelay,${EXTEN},1\(${UserB\_DeskPhone},9,600\)\)

exten =&gt; owner,1,Gosub\(subDialDelay,${EXTEN},1\(${UserB\_SoftPhone},12,18\)\)

**Note**

Even though the destination for a local channel is really just dialplan—the same as you might jump to with a Goto\(\)—these constructs tend to be very special-purpose, and fit into the dialplan better in their own area, down with the subroutines. That’s why we named the context with the prefix local. It’s not required, but makes things easier to make sense of.

Now we stitch it all together in our \[sets\] context.

First, let’s provide a way to dial each local channel individually, so we can sanity check each one to be sure it’s doing what it should.

exten =&gt; 103,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_SoftPhone},${EXTEN},default,24\)\)

; These are for testing individually before we put them together

exten =&gt; 104,1,Dial\(Local/receptionist@localDialDelay\)

exten =&gt; 105,1,Dial\(Local/team\_one@localDialDelay\)

exten =&gt; 106,1,Dial\(Local/team\_two@localDialDelay\)

exten =&gt; 107,1,Dial\(Local/owner@localDialDelay\)

Finally, let’s deliver the finished product.

exten =&gt; 107,1,Dial\(Local/owner@localDialDelay\)

;We're going to assign some variables in order to

;keep the dial string easier to read

exten =&gt; 108,1,Noop\(DialDelay\)

 same =&gt; n,Set\(Recpn=Local/receptionist@localDialDelay\)

 same =&gt; n,Set\(Team1=Local/team\_one@localDialDelay\)

 same =&gt; n,Set\(Team2=Local/team\_two@localDialDelay\)

 same =&gt; n,Set\(Boss=Local/owner@localDialDelay\)

 same =&gt; n,Dial\(${Recpn}&${Team1}&${Team2}&${Boss},600\)

You really need to register a few phones and try this out, to see it all come together.

The solution we have created here is perfect for learning about local channels, but it has a few problems that need to be understood if you ever want to put it into production:

* Even though we have set a dial timeout, you will find that SIP endpoints have minds of their own. It’s not uncommon for a SIP endpoint to have its own ideas about timeout. So, you might set it to ring for 600 seconds, and wonder why it drops the call after a minute or so. You could spend hours troubleshooting your dialplan, only to discover the problem was a setting at the other end. Test each piece of the solution before you glue them all together.
* Cell phones have their own voicemail, and if that answers the call, Asterisk will connect the call to that “answered” channel. One way around this is to hang up before that happens, and then call immediately back. It’s ugly though, and not recommended.
* Cell phones will often go immediately to a voicemail if they’re out of range or turned off. That counts as an answer as far as Asterisk is concerned. This solution does not handle that.
* Call setup to a cell phone \(i.e., the time between when you dial and when it starts ringing\) typically takes a dozen seconds or so.
* Remember that a softphone on a cell phone is not at all the same as a phone call to that cell phone. One is a SIP connection, the other is a PSTN call. \(You can actually ring both at the same time if you want, but that’s not necessarily a good idea.\)
* Some types of smartphones will give priority to incoming GSM calls. If you are on a call on the softphone, and somebody calls your cell number, the softphone may get put on hold. Different phones handle this differently.
* We haven’t really handled overflow here. What happens if nobody answers? It doesn’t matter in the lab, but you can be sure it’ll matter in a production environment.
* Dial\(\) expects ringing back from the destination. If all of your local channels have a Wait\(\) delay, the caller will hear silence until something indicates ringing. You can fix this by having Dial\(\) fake the ringing with the 'r' option, or by adding a dummy local channel that just returns ringing.

**Note**

If you check the sample dialplan, we’ve added a solution to the silence problem on delayed local channels

That’s it. Local channels: build them piece-by-piece and you’ll be delivering a powerful dialplan in no time.

They’re incredibly useful when building complex queueing applications as well.

## Using the Asterisk Database

Asterisk provides a simple mechanism for storing data called the Asterisk database \(AstDB\). This is not an external relational database, but simply an SQLite-based backend for storing simple key/value pairs.

The Asterisk database stores its data in groupings called families, with values identified by keys. Within a family, a key may be used only once. For example, if we had a family called test, we could store only one value with a key called count. Each stored value must be associated with a family.

### Storing Data in the AstDB

To store a new value in the Asterisk database, we use the Set\(\) application with the DB\(\) function. For example, to assign the count key in the test family with the value of 1, we would write the following:

exten =&gt; 216,1,NoOp\(\)

 same =&gt; n,Set\(DB\(testkey/count\)=1\)

Make a test call to 216 to set the value. Note that if a key named count already exists in the test family, its value will be overwritten with the new value \(in this case, the value is hardcoded so obviously will get overwritten with the same value, but later we’ll see how we can change the value, and have that stored\).

You can also store values from the Asterisk command line, by running the command database put family key value. For our example, you would type database put test count 1.

So, while we’re at it, let’s also plug a value into the database from the console:

\*CLI&gt; database put somekey somevalue 42

Let’s query the database from the console to see what values are in there:

\*CLI&gt; database show

If all is well, you should see output similar to the following:

/pbx/UUID : d562019a-d2c4-4b88-bcd9-602b3b46fe07

/somekey/count : 1

/somekey/somevalue : 42

/testkey/count : 1

4 results found.

localhost\*CLI&gt;

### Retrieving Data from the AstDB

To retrieve a value from the Asterisk database and assign it to a variable, we will again use the Set\(\) application and the DB\(\) function. Let’s retrieve the value of somevalue \(from the somekey family\), assign it to a variable called THE\_ANSWER, and then speak the value to the caller:

exten =&gt; 217,1,NoOp\(\)

 same =&gt; n,Set\(THE\_ANSWER=${DB\(somekey/somevalue\)}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayNumber\(${THE\_ANSWER}\)

You may also check the value of a given key from the Asterisk command line by running the command database get family key. To view the entire contents of the AstDB, use the database show command.

### Deleting Data from the AstDB

There are two ways to delete data from the Asterisk database. To delete a key, you can use the DB\_DELETE\(\) application. It takes the path to the key as its arguments, like this:

; deletes the key and returns its value in one step

exten =&gt; 218,1,Verbose\(0, We just blew away ${DB\_DELETE\(somekey/somevalue\)}\)

You can also delete an entire key family by using the DBdeltree\(\) application. The DBdeltree\(\) application takes a single argument: the name of the key family to delete. To delete the entire test family, do the following:

exten =&gt; 219,1,DBdeltree\(somekey\)

To delete keys and key families from the AstDB via the command-line interface, use the database del key and database deltree family commands, respectively.

If you call extension 217 now, you will see that there is nothing said, because nothing is returned by the database. You can also run database show from the CLI, and note that that family and key have been removed.

### Using the AstDB in the Dialplan

There are an infinite number of ways to use the Asterisk database in a dialplan. To introduce the AstDB, we’ll look at two simple examples. The first is a simple counting example to show that the Asterisk database is persistent \(it even survives system reboots\). In the second example, we’ll use the BLACKLIST\(\) function to evaluate whether or not a number is on the blacklist and should be blocked.

To begin the counting example, let’s first retrieve a number \(the value of the count key\) from the database and assign it to a variable named COUNT. If the key doesn’t exist, DB\(\) will return NULL \(no value\). Therefore, we can use the ISNULL\(\) function to verify whether or not a value was returned. If not, we will initialize the AstDB with the Set\(\) application, where we will set the value in the database to 1. This will only happen if the database entry does not exist:

exten =&gt; 220,1,NoOp\(\)

 same =&gt; n,Set\(COUNT=${DB\(test/count\)}\) ; retrieve current value in database

 same =&gt; n,GotoIf\($\[${ISNULL\(${COUNT}\)}\]?firstcount:saycount\) ; is there a value?

 same =&gt; n\(firstcount\),Set\(DB\(test/count\)=1\) ; set the value to 1

 same =&gt; n,Goto\(saycount\)

 same =&gt; n\(saycount\),NoOp\(\)

 same =&gt; n,Answer

 same =&gt; n,SayNumber\(${COUNT}\)

 same =&gt; n,Goto\(increment\) ; not reqd but a good habit

 same =&gt; n\(increment\),Set\(COUNT=$\[${COUNT} + 1\]\) ; increment by one

 same =&gt; n,Set\(DB\(test/count\)=${COUNT}\) ; and assign new value to database

 same =&gt; n,Goto\(saycount\) ; loop back and say it again

Test this out. Listen to it count for a while, and then hang up. When you dial this extension again, it will continue counting from where it left off. The value stored in the database will be persistent, even across a restart of Asterisk.

In the early days of Asterisk, the built-in database was essential. Today, however, it’s not as commonly used. It’s probably good for setting a few semaphores here and there, but for the most part, if you want to store data, use one of the relational database backends \(we discuss relational database integration in later chapters\).

## Handy Asterisk Features

Now that we’ve gone over some more of the basics, let’s look at a few popular functions that have been incorporated into Asterisk.

### Conferencing with ConfBridge\(\)

The ConfBridge\(\) application allows multiple callers to converse together, as if they were all in the same physical location. Some of the main features include:

* The ability to create password-protected conferences
* Conference administration \(mute conference, lock conference, or kick off participants\)
* The option of muting all but one participant \(useful for company announcements, broadcasts, etc.\)
* Static or dynamic conference creation
* High-definition audio that can be mixed at sample rates ranging from 8 kHz to 96 kHz
* Video capabilities, including the addition of dynamically switching video feeds based on loudest talker
* Dynamically controlled menu system for both conference administrators and users
* Additional options available in the confbridge.conf configuration file

In this chapter we are focused on the dialplan, so we’re only going to demonstrate a basic audio conference bridge:

$ sudo -u asterisk vim /etc/asterisk/confbridge.conf

\[general\]

\[default\_user\]

type=user

\[default\_bridge\]

type=bridge

After building the confbridge.conf file, we need to load the app\_confbridge.so module. This can be done at the Asterisk console:

\*CLI&gt; module load app\_confbridge.so

With the module loaded, we can build a simple dialplan to access our conference bridge:

exten =&gt; 221,1,NoOp\(\)

 same =&gt; n,ConfBridge\(${EXTEN}\)

This is just the tip of the iceberg for conferencing. We’ve got the base configuration done, but there is much more functionality to be configured. We’ll cover it in a little more detail in [Chapter 11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch11.html%22%20/l%20%22asterisk-SysAdmin).

## Handy Dialplan Functions

We discussed functions earlier in this chapter, but there’s more to say. There are currently around 150 dialplan functions provided by the Asterisk dialplan. Here is a small, curated list of a few worth experimenting with.

### CALLERID\(\)

CALLERID\(\) supports many different datatypes, but you’ll find that you’ll typically use one of name or num.

exten =&gt; 222,1,Noop\(CALLERID function\)

 same =&gt; n,Noop\(CALLERID currently ${CALLERID\(all\)}\)

 same =&gt; n,Set\(CALLERID\(num\)=4169671111\)

 same =&gt; n,Noop\(CALLERID now ${CALLERID\(all\)}\)

 same =&gt; n,Set\(CALLERID\(name\)="Somename"\)

 same =&gt; n,Noop\(CALLERID now ${CALLERID\(all\)}\)

 same =&gt; n,Hangup\(\)

Don’t worry about the rest of them. If you need ’em, you’ll know what they are or why you want to use them.

### CHANNEL\(\)

CHANNEL\(\) allows you to interact with an absolute boatload of data relating to the channel. Some items allow you to modify them, while others will only be useful for reference \(for example, peerip will allow you to read, but not change, the IP address of the peer\). There are also channel variables that only work with certain channel types \(for example, pjsip items can of course only be used on PJSIP channels\).

exten =&gt; 223,1,Noop\(CHANNEL function\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Noop\(CHANNEL\(name\) is ${CHANNEL\(name\)}\)

 same =&gt; n,Noop\(CHANNEL\(musicclass\) is ${CHANNEL\(musicclass\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_jitter\) is ${CHANNEL\(rtcp,all\_jitter\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_loss\) is ${CHANNEL\(rtcp,all\_loss\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,all\_rtt\) is ${CHANNEL\(rtcp,all\_rtt\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,txcount\) is ${CHANNEL\(rtcp,txcount\)}\)

 same =&gt; n,Noop\(CHANNEL\(rtcp,rxcount\) is ${CHANNEL\(rtcp,rxcount\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,local\_uri\) is ${CHANNEL\(pjsip,local\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,remote\_uri\) is ${CHANNEL\(pjsip,remote\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,request\_uri\) is ${CHANNEL\(pjsip,request\_uri\)}\)

 same =&gt; n,Noop\(CHANNEL\(pjsip,local\_tag\) is ${CHANNEL\(pjsip,local\_tag\)}\)

### CURL\(\)

CURL\(\) is a simple yet powerful function that provides a one-liner method for resolving URLs, which in many cases is all you need for a basic interaction with an external web service.

exten =&gt; 224,1,Noop\(CURL function\)

 same =&gt; n,Set\(ExternalIP=${CURL\(http://whatismyip.akamai.com\)}\)

 same =&gt; n,Noop\(The external IP address is ${ExternalIP}\)

If you need a more complex interaction with an external service, it could be that you are going to want an AGI program of some sort. Still, you can embed a ton of data in a URL, and for simplicity, CURL\(\) is hard to beat.

### CUT\(\)

If you need to slice-and-dice your variables, you’ll find the CUT\(\) function essential. The form is simple:

CUT\(varname,char-delim,range-spec\)

It can be visually tricky, as the delimiter character can be difficult to see nested in between two commas \(for example, if the delimiter was a dot/decimal/period\). Let’s expand on the previous example to see what it’s good for \(and give you a visual example of how the delimiter can get lost in the syntax\).

exten =&gt; 225,1,Noop\(CUT function\)

 same =&gt; n,Set\(ExternalIP=${CURL\(http://whatismyip.akamai.com\)}\)

 same =&gt; n,Noop\(The external IP address is ${ExternalIP}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,1\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,2\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,3\)}\)

 same =&gt; n,Playback\(letters/dot\)

 same =&gt; n,SayDigits\(=${CUT\(ExternalIP,.,4\)}\)

**Note**

Note that you call the CUT\(\) function with the braces ${CUT\(\)}, but the variable being referenced inside CUT\(\) is defined without the braces. This is because we are naming the variable, not asking for its contents \(CUT\(\) will deal with the contents, so we just need to name the variable it will be slicing and dicing, and it will dive into what is stored there\).

### IF\(\) and STRFTIME\(\)

The combination of IF\(\) and STRFTIME\(\) is a powerful construct, and you will find it an essential part of your dialplan logic:

exten =&gt; 226,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Playback\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2\] = 1\]?hear-odd-noise:good-evening\)}\)

Wait...what?[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406549320)

Let’s break this down \(we’re going to indent the code in an odd manner in order to show the progression of the nested functions and operators\):

exten =&gt; 227,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\) ; current time - just seconds

 same =&gt; n,Noop\($\[ ${STRFTIME\(,,%S\)} % 2 \]\) ; divide by 2 - return remainder

 same =&gt; n,Noop\(${IF\($\[ $\[ ${STRFTIME\(,,%S\)} % 2 \] = 1 \]?odd:even\)}\)

same =&gt; n,Playback\(${IF\($\[ $\[ ${STRFTIME\(,,%S\)} % 2 \] = 1 \]?hear-odd-noise:good-evening\)}\)

The IF\(\) function allows us to pass logic to the Playback\(\) application. We’re effectively saying, “If it’s true that the time, in seconds, is odd, play the hear-odd-noise prompt, otherwise, play the good-evening prompt.”

If we line up the code in a more typical fashion, it looks like this \(note that some of the optional spaces have also been removed\):

exten =&gt; 228,1,Noop\(IF\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Wait\(.5\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\)

 same =&gt; n,Noop\($\[${STRFTIME\(,,%S\)} % 2\]\)

 same =&gt; n,Noop\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2 \] = 1\]?odd:even\)}\)

 same =&gt; n,Playback\(${IF\($\[$\[${STRFTIME\(,,%S\)} % 2 \] = 1\]?hear-odd-noise:good-evening\)}\)

The final line is very difficult to read unless you know how we got there, but it demonstrates the power of nesting.

At first these constructs may seem difficult to write, so break them down and perform them line by line, and eventually they’ll get easier \(and your dialplan will subsequently become more powerful\). Play with them.

### LEN\(\)

Being able to return the length of something with the LEN\(\) function can be very handy.

exten =&gt; 229,1,Noop\(LEN\)

 same =&gt; n,Set\(LengthyString=${RAND\(1,2000\)}\)

 same =&gt; n,Noop\(${LEN\(${LengthyString}\)}\)

 same =&gt; n,Noop\(${IF\( $\[ ${LEN\(${LengthyString}\)} &lt;= 3 \]?tooshort:youcanride\)}\)

### REGEX\(\)

Yes, you can use regular expressions within Asterisk. This is a somewhat advanced topic, not because REGEX\(\) is a complicated function in itself, but because regular expressions are a study in themselves.

Check out [http://www.regular-expressions.info/](http://www.regular-expressions.info/) for more info, or grab a copy of O’Reilly’s Mastering Regular Expressions by Jeffrey E. F. Friedl.

Get used to using other functions in Asterisk, get some experience with regular expressions, and then give REGEX\(\) a try.

### STRFTIME\(\)

We just saw the STRFTIME\(\) function in our IF\(\) example. It allows you to return a time in various formats. In general, you want the input to be empty \(which defaults to the current time\). You can also give this function a specific Unix epoch string and it’ll work from that.

exten =&gt; 230,1,Noop\(STRFTIME\)

 same =&gt; n,Noop\(${STRFTIME\(,,%S\)}\) ; we've seen this before

 same =&gt; n,Noop\(${STRFTIME\(,,%B\)}\) ; month

 same =&gt; n,Noop\(${STRFTIME\(,,%H\)}\) ; hour in 24hr format

 same =&gt; n,Noop\(${STRFTIME\(,,%m\)}\) ; month as a decimal

 same =&gt; n,Noop\(${STRFTIME\(,,%M\)}\) ; minute

 same =&gt; n,Noop\(${STRFTIME\(,,%Y\)}\) ; year - 4 digits

 same =&gt; n,Noop\(${STRFTIME\(,,%Y-%m-%d %H:%m:%S\)}\) ; string some together

## Conclusion

In this chapter, we’ve covered a few more of the many applications in the Asterisk dialplan, and hopefully we’ve given you some more tools that you can use to further experiment with creating your own dialplans. As with other chapters, we invite you to go back and reread any sections that require clarification.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-1-marker) Remember that when you reference a variable you can call it by its name, but when you refer to a variable’s value, you have to use the dollar sign and brackets around its name.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-2-marker) For more on regular expressions, grab a copy of the ultimate reference, Jeffrey E. F. Friedl’s [Mastering Regular Expressions](http://shop.oreilly.com/product/9780596528126.do) \(O’Reilly, 2006\), or visit [http://www.regular-expressions.info](http://www.regular-expressions.info/).

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-CHP-6-FN-3-marker) If you don’t know what a ^ has to do with regular expressions, you simply must read [Mastering Regular Expressions](http://shop.oreilly.com/product/9780596528126.do). It will change your life!

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406809256-marker) If you want to test this \(which you do\), you can pick one of your working lab devices, and in the asterisk database, under the ps\_endpoints table, set the callerid field to '8885551212'. Then you can make a call from it to 214 to see the block in action.

UPDATE asterisk.ps\_endpoints SET callerid='8885551212' WHERE id='&lt;endpoint you chose as the victim&gt;'

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406801064-marker) But we do it this way because it’s easier to read.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406763720-marker) We have no idea how to implement Easter, but are open to suggestions.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406696296-marker) Obsolete Android phones and tablets can be great for this.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22idm46178406549320-marker) There is a C language function named STRFTIME\(\) that returns the current time as a formatted string. This works similarly to that. In fact, the format portion of the function takes the exact same syntax as the C function.

