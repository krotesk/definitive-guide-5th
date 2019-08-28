---
description: Основы диалплана
---

# Глава 6

> _Всё должно быть изложено так просто, как только возможно, но не более того._ 
>
> — Альберт Эйнштейн

Диалплан-это сердце вашей системы Asterisk. Он определяет, как звонки поступают в систему и выходят из нее. Диалплан является скриптовым языком, который определяет инструкции, которым Asterisk следует в ответ на вызовы, поступающие от каналов. В отличие от традиционных телефонных систем, диалплан Asterisk полностью настраивается.

Опытные разработчики программного обеспечения считают код диалплана Asterisk архаичным и часто предпочитают управлять потоком вызовов с помощью API Asterisk, таких как AMI и ARI \(которые мы обсудим в последующих главах\). Независимо от ваших планов в этом отношении, изучение поведения Asterisk намного проще, если вы сначала поймете диалплан. Возможно, также стоит отметить, что диалплан Asterisk настроен на производительность и поэтому является самым быстрым способом выполнения потока вызовов с точки зрения быстродействия и минимальной нагрузки на систему. Диалплан работает быстро.

В этой главе представлены основные понятия диалплана, которые лягут в основу любого диалплана, который вы пишете. Не пропускайте слишком много из этой главы, так как примеры строятся друг на друге, и это так принципиально важно для Asterisk. Обратите также внимание, что эта глава ни в коем случае не является исчерпывающим обзором всех возможных вещей, которые может сделать диалплан; наша цель - охватить только самое необходимое. В последующих главах мы рассмотрим более сложные темы диалплана. Вам рекомендуется экспериментировать.

## Синтаксис диалплана

Диалплан Asterisk задается в конфигурационном файле с именем _extensions.conf_, расположенном в каталоге _/etc/asterisk_.

Структура диалплана состоит из четырех иерархических компонентов: контекстов \(Context\), расширений \(Extension\), приоритетов \(Priority\) и приложений \(Application\) \(смотри Рисунок[ 6-1](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig0601)\).

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 6-1. &#x418;&#x435;&#x440;&#x430;&#x440;&#x445;&#x438;&#x44F; &#x434;&#x438;&#x430;&#x43B;&#x43F;&#x43B;&#x430;&#x43D;&#x430;](.gitbook/assets/0%20%281%29.png)

Давайте нырнем прямо туда.

{% hint style="success" %}
**Примеры файлов конфигурации**

А основной файл _extensions.conf_ был создан как часть процесса установки ранее в этой книге. Мы будем опираться на этот файл в этой главе.

Asterisk также поставляется с подробным файлом _extensions.conf,_ который может быть установлен с образцами файлов конфигурации \(команда установки `make samples` сделает это\), и если вы запустили эту команду \(которую мы не рекомендуем во время установки, но предлагается установщиком\), у вас, скорее всего, будет файл _/etc/asterisk/extensions.conf_, который переполнен информацией. Вместо того, чтобы начинать с примера файла, мы предлагаем вам построить свой _extensions.conf_ с нуля с пустым файлом \(вы можете переименовать или переместить его куда-нибудь, если хотите сохранить в качестве ссылки\).

Как и говорилось файл примера _extensions.conf_ - это фантастический ресурс, полный примеров и идей, которые вы можете использовать после того, как изучили основные понятия. Если вы выполнили наши инструкции по установке, то найдете файл _extensions.conf.sample_ в каталоге _~/src/asterisk-15.&lt;TAB&gt;/configs/samples_ \(наряду со многими другими образцами файлов конфигурации\).
{% endhint %}

\*\*\*\*

### Контексты

Диалплан делится на разделы, называемые контекстами, которые служат для разделения различных частей диалплана. Расширение, определенное в одном контексте, полностью изолировано от расширений в любом другом контексте, если взаимодействие специально не разрешено.

В качестве простого примера представим, что у нас есть две компании, совместно использующие сервер Asterisk. Если мы поместим каждого автосекретаря компании \(IVR\) в свой собственный контекст, две компании будут полностью отделены друг от друга. Это позволяет нам самостоятельно определить, что происходит, когда, скажем, набирается номер 0:

* Абоненты, набирающие 0 из голосового меню компании A, должны быть переданы администратору компании A.
* Абоненты, набравшие 0 в голосовом меню компании B, будут отправлены в отдел обслуживания клиентов компании B.

Оба абонента находятся в одной и той же системе, взаимодействуя с одной и той же абонентской группой, но поскольку они прибыли в разные контексты, то испытывают совершенно разные потоки вызовов. То, что происходит с каждым входящим вызовом, определяется кодом диалплана в каждом контексте.

{% hint style="info" %}
**Примечание**

Это очень важное соображение. С традиционными УАТС, как правило, существует набор значений по умолчанию для таких вещей, как прием, что означает, что если вы забудете их определить, они, вероятно, будут работать в любом случае. В Asterisk все наоборот. Если вы не скажете Asterisk, как обрабатывать каждую ситуацию, и он столкнется с чем-то, что не может обработать, вызов, как правило, будет отклонен.
{% endhint %}

Контексты определяются в файле _extensions.conf_, помещая имя контекста в квадратные скобки \(\[\]\). Имя может состоять из букв A - Z \(верхний и нижний регистр\), чисел от 0 до 9, а также дефиса и подчеркивания.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#asterisk-DP-Basics-FN-1) Контекст для входящих вызовов от оператора связи может быть назван так:

```text
[incoming]
```

{% hint style="info" %}
**Примечание**

Имена контекстов имеют максимальную длину 79 символов \(80 символов минус 1 завершающий null\).
{% endhint %}

Или возможно:

```text
[incoming_company_A]
```

Что тогда, конечно, может потребовать что-то вроде:

```text
[incoming_company_B]
```

Все инструкции, помещенные после определения контекста, являются частью этого контекста, пока не будет определен следующий контекст.

В начале диалплана есть два специальных раздела с именами `[general]` и `[globals]`. Раздел `[general]` содержит список общих настроек абонентской группы \(о которых вам, вероятно, никогда не придется беспокоиться\), а контекст `[globals]`мы вскоре обсудим. На данный момент важно только знать, что эти две метки не являются контекстами, несмотря на использование синтаксиса контекста. Не используйте `[general]`, `[globals]` и `[default]`[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#idm46178408408152) в качестве имен контекста, но в противном случае называйте свои контексты как угодно.

Контексты в типичном файле _extensions.conf_ могут быть структурированы примерно так:

```text
[general] ; он всегда должен быть здесь
[globals] ; глобальные переменные (мы обсудим их позже)
[incoming] ; звонки от поставщиков услуг могут поступать сюда
[sets] ; в простой системе мы можем использовать это
[sets1] ; многопользовательская же нуждается в этом (устройства от одной компании вводят диалплан здесь)
[sets2] ; ... и этом (другая группа устройств может войти в диалплан здесь)
[services] ; специальные услуги, такие как конференц-связь, могут быть определены здесь
```



Когда вы определяете канал \(что не делается в _extensions.conf_\), одним из обязательных параметров в каждом определении канала является `context`. _Контекст - это точка в диалплане, куда будут поступать соединения из этого канала._ Таким образом, способ подключения канала к диалплану осуществляется через контекст \(Рисунок 6-2\).

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 6-2. &#x421;&#x432;&#x44F;&#x437;&#x44C; &#x43C;&#x435;&#x436;&#x434;&#x443; &#x43A;&#x43E;&#x43D;&#x444;&#x438;&#x433;&#x443;&#x440;&#x430;&#x446;&#x438;&#x435;&#x439; &#x43A;&#x430;&#x43D;&#x430;&#x43B;&#x430; \(&#x441;&#x43B;&#x435;&#x432;&#x430;\) &#x438; &#x43A;&#x43E;&#x43D;&#x442;&#x435;&#x43A;&#x441;&#x442;&#x430;&#x43C;&#x438; &#x432; &#x434;&#x438;&#x430;&#x43B;&#x43F;&#x43B;&#x430;&#x43D;&#x435; \(&#x441;&#x43F;&#x440;&#x430;&#x432;&#x430;\)](.gitbook/assets/1.png)

{% hint style="info" %}
**Примечание**

Это одна из наиболее важных концепций для понимания при работе с каналами и абонентскими группами. Устранение неполадок потока вызовов намного проще, если вы понимаете связь между контекстом канала \(который сообщает каналу, где подключаться к диалплану\) и контекстом диалплана \(где мы создаем поток вызовов, который происходит при поступлении вызова\).
{% endhint %}

Важным \(возможно, самым важным\) использованием контекстов является обеспечение конфиденциальности и безопасности. При правильном использовании контекстов можно предоставить некоторым каналам доступ к функциям \(например, междугородним вызовам\), которые недоступны другим. Если вы не проработаете свою абонентскую группу тщательно, то можете непреднамеренно позволить другим использовать вашу систему в корыстных целях. Пожалуйста, имейте это в виду, когда строите свою систему Asterisk; в интернете есть много ботов, которые были специально написаны для идентификации и использования плохо защищенных систем Asterisk.

{% hint style="danger" %}
**Предупреждение**

[Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Important+Security+Considerations) описывает несколько шагов, которые вы должны предпринять, чтобы сохранить вашу систему Asterisk в безопасности. Жизненно важно чтобы вы прочитали и поняли эту страницу. Если вы игнорируете меры безопасности, изложенные там, то можете в конечном итоге позволить всем и каждому совершать междугородние или платные звонки за ваш счет!

Если вы не относитесь к безопасности вашей системы Asterisk серьезно, то можете в конечном итоге поплатиться буквально. _Пожалуйста_, потратьте время и усилия, чтобы защитить вашу систему от мошенничества.
{% endhint %}

### Extensions \(расширения\)

В телекоммуникационной отрасли слово _extension \(расширение\)_ обычно относится к числовому идентификатору, который при наборе будет звонить на телефон \(или вызывать системный ресурс, такой как голосовая почта или очередь\). В Asterisk расширение представляет нечто гораздо более мощное, поскольку оно определяет уникальную серию шагов \(каждый шаг, содержащий приложение\), через которые Asterisk будет принимать этот вызов.

В каждом контексте мы можем определить столько \(или несколько\) расширений, сколько потребуется. Когда определенное расширение запускается \(входящим каналом\), Asterisk будет следовать шагам, определенным для этого расширения. Поэтому именно расширения определяют, что происходит с вызовами, когда они проходят через диалплан. Хотя расширения могут использоваться для указания телефонных добавочных номеров в традиционном смысле \(т.е. расширение 153 вызовет звонок SIP-телефона на столе Джона\), в диалплане Asterisk они могут использоваться для гораздо большего.

Синтаксис расширения - это слово `exten`, за которым следует стрелка, образованная знаком равенства и знаком больше, как это:

```text
exten =>
```

После этого следует имя \(или номер\) расширения.

При работе с традиционными телефонными системами мы склонны думать о расширениях как о номерах, которые вы набираете, чтобы сделать еще один телефонный звонок. В Asterisk имена расширений могут быть любыми комбинациями цифр и букв. В этой и следующей главах мы будем использовать как цифровые, так и буквенно-цифровые расширения.

{% hint style="warning" %}
**Совет**

Назначение имен для расширений может показаться необычной концепцией, но когда вы понимаете, что SIP поддерживает набор всех видов комбинаций символов \(все, что является допустимым URI, строго говоря\), это имеет смысл. Это одна из особенностей, которая делает Asterisk настолько гибким и мощным.
{% endhint %}

Каждый шаг расширения состоит из трех компонентов:

* Имя \(или номер\) расширения
* Приоритет \(каждое расширение может включать в себя несколько шагов; номер шага называется "приоритет”\)
* Приложение \(или команда\), которое будет выполняться на этом шаге

Эти три компонента разделены запятыми, как это:

```text
exten => name,priority,application()
```

Вот простой пример:

```text
exten => 123,1,Answer()
```

Имя расширения - `123`, приоритет - `1`, а приложение - `Answer()`.

### Приоритеты

Каждое расширение может иметь несколько шагов, называемых _приоритетами_. Приоритеты нумеруются последовательно, начиная с 1 и каждый выполняет одно конкретное приложение. Например, следующий добавочный номер ответит на звонок с приоритетом номер 1, а затем повесит трубку с приоритетом номер 2. Шаги в расширении происходят один за другим.

```text
exten => 123,1,Answer()
exten => 123,2,Hangup()
```

Совершенно очевидно, что этот код на самом деле не делает ничего полезного. Ключевым моментом здесь является то, что для конкретного расширения Asterisk следует за приоритетами по порядку.

```text
exten => 123,1,Answer()
exten => 123,2,делаем что-то
exten => 123,3,делаем что-то ещё
exten => 123,4,сделаем ещё одну вещь
exten => 123,5,Hangup()
```

Этот стиль синтаксиса диалплана все еще встречается время от времени, хотя \(как вы вскоре увидите\) он обычно больше не используется для нового кода. Более новый синтаксис похож, но упрощен.

#### Ненумерованные приоритеты

В старых версиях Asterisk нумерация приоритетов вызывала много проблем. Представьте себе расширение, которое имеет 15 приоритетов, а затем нужно что-то добавить на Шаге 2: все последующие приоритеты должны быть перенумерованы вручную. Asterisk не обрабатывает пропущенные шаги или неправильно пронумерованные приоритеты, и отладка этих типов ошибок была разочаровывающей.

Начиная с версии 1.2 Asterisk решил эту проблему: он ввел использование приоритета `n`, который означает “next." Каждый раз, когда Asterisk встречает приоритет с именем `n`, он принимает номер предыдущего приоритета и добавляет 1. Это упрощает внесение изменений в ваш диалплан, так как вам не нужно постоянно перенумеровывать все ваши шаги. Например, ваш диалплан может выглядеть примерно так:

```text
exten => 123,1,Answer()
exten => 123,n,do something
exten => 123,n,do something else
exten => 123,n,do one last thing
exten => 123,n,Hangup()
```

Внутри Asterisk будет вычислять следующий номер приоритета каждый раз, когда он сталкивается с `n`.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#idm46178408333272) Теперь, если мы хотим добавить новый элемент в приоритет 3, мы просто вводим новую строку, где она нам нужна, и не требуется перенумерация.

```text
exten => 123,1,Answer()
exten => 123,n,do something
exten => 123,n,SOME NEW THING
exten => 123,n,do something else
exten => 123,n,do one last thing
exten => 123,n,Hangup()
```

Имейте в виду, что вы всегда должны указывать приоритет № 1. Если вы случайно поставили `n` вместо `1` для первого приоритета \(распространенная ошибка даже среди опытных кодеров диалплана\), вы обнаружите после перезагрузки диалплана, что расширение не будет существовать.

#### Оператор same =&gt;

Для дальнейшего упрощения написания диалплана был создан новый синтаксис. Пока расширение остается неизменным, вы можете просто ввести `same =>` с последующим приоритетом и приложением, а не вводить полное расширение в каждой строке:

```text
exten => 123,1,Answer()
 same => n,do something
 same => n,do something
 same => n,do one last thing
 same => n,Hangup()
```

Этот стиль диалплана также облегчит копирование кода из одного расширения в другое. Это предпочтительный и рекомендуемый стиль. Единственная причина обсуждения предыдущих стилей - помочь понять, как мы сюда попали.

{% hint style="success" %}
Не ошибитесь, диалплан Asterisk весьма своеобразен. Многие люди избегают его вообще, и использовать AGI и ARI, чтобы написать свой диалплан.

Хотя, конечно, есть что сказать для написания диалплана на внешнем языке \(и мы рассмотрим его в последующих главах\), диалплан Asterisk является родным для него, и вы не получите лучшей производительности чем c ним. Код диалплана выполняется быстро.

Кроме того, если вы хотите понять, как Asterisk думает, вам нужно понять его диалплан.
{% endhint %}

#### Метки приоритетов

Метки приоритетов позволяют назначить имя приоритету в пределах расширения. Это должно гарантировать что вы можете ссылаться на приоритет иначе чем его номер \(который, вероятно, неизвестен, учитывая, что диалпланы теперь, как правило, используют ненумерованные приоритеты\). Позже вы узнаете, что часто необходимо отправлять вызовы из других частей диалплана на определенный приоритет в определенном расширении. Чтобы назначить текстовую метку приоритету, просто добавьте метку в скобках после приоритета, например:

```text
exten => 123,n(label),application()
```

Позже мы рассмотрим, как переключаться между различными приоритетами на основе логики диалплана. Вы увидите гораздо больше меток приоритетов и будете чаще использовать их в своих диалпланах.

{% hint style="danger" %}
**Предупреждение**

Очень распространенной ошибкой при написании меток является вставка запятой между then и \(, например:

`exten => 555,n,(label),application() ;<-- ЭТО НЕ БУДЕТ РАБОТАТЬ`

`exten => 556,n(label),application() ;<-- Это, что надо`

Эта ошибка нарушит часть вашего диалплана и вы получите сообщение об ошибке, указывающее, что приложение не может быть найдено.
{% endhint %}

### Приложения

Приложения — это рабочие лошадки диалплана. Каждое приложение выполняет определённое действие в текущем канале, такое как — воспроизведение звука, приём набора сигналов DTMF, поиск чего-то в базе данных, выполнение вызова в канал, завершение вызова, кормление кошки или что-то иное.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#idm46178408295288) В предыдущем примере мы показали два простых приложения: `Answer()` и `Hangup()`. Очевидно что они делают, но также очевидно что сами по себе они не очень полезны.

Некоторые приложения, включая `Answer()` и `Hangup()` не требуют дополнительных инструкций для выполнения своей задачи. Но большинству приложений требуется дополнительная информация. Эти дополнительные элементы или аргументы передаются в приложения чтобы повлиять на выполнение действий. Чтобы передать аргументы приложению, поместите их между круглыми скобками, которые следуют за именем приложения, разделяя запятыми. 

### Приложения Answer\(\), Playback\(\) и Hangup\(\)

Приложение `Answer()` используется для ответа на канал, который звонит. Это кажется простой вещью, но много вещей происходит на канале с этой одной командой. `Answer()` сообщает каналу отправить обратно на дальний конец сообщение, что вызов был отвечен, а также включить медиа-пути \(сетевые потоки, которые будут нести звук между вызывающим абонентом и системой\). Как мы уже упоминали ранее, `Answer()` не принимает аргументов. `Answer()` не всегда требуется \(на самом деле, в некоторых случаях он может быть вообще нежелательным\), но это эффективный способ обеспечить подключение канала перед выполнением дальнейших действий.

{% hint style="success" %}
**Приложение Progress\(\)**

Иногда полезно иметь возможность передавать информацию обратно в сеть перед ответом на вызов. Приложение `Progress()` пытается предоставить информацию о ходе выполнения вызова исходному каналу. Некоторые операторы связи ожидают этого, и таким образом вы можете решить странные проблемы с сигнализацией, вставив `Progress()` в диалплан, куда поступают ваши входящие вызовы. С точки зрения биллинга, использование `Progress()` позволяет поставщику услуг знать, что вы обрабатываете вызов, не запуская счетчик биллинга.
{% endhint %}

Приложение `Playback()` используется для воспроизведения ранее записанного звукового файла по каналу. Ввод от пользователя игнорируется, что означает невозможность использования `Playback()` в автосекретаре, например если не хотите принимать ввод в этот момент.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#idm46178408274728)

{% hint style="info" %}
**Подсказка**

Asterisk поставляется со многими профессионально записанными звуковыми файлами, которые должны быть найдены в каталоге звуков по умолчанию \(обычно _/var/lib/asterisk/sounds_\). При компиляции Asterisk можно установить различные наборы образцов звуков, записанных на различных языках и в различных форматах файлов. Мы будем использовать эти файлы во многих наших примерах. Некоторые из файлов в наших примерах взяты из дополнительного звукового пакета, который мы установили в [Главе 3](glava-03.md). Вы также можете иметь свои собственные звуковые подсказки, записанные в тех же голосах, что и стоковые подсказки, посетив [www.theivrvoice.com](www.theivrvoice.com). Далее в книге мы поговорим о том, как можно использовать телефон и абонентскую группу для создания и управления собственными системными записями \(или импорта _.wav_ файлов\).
{% endhint %}

Чтобы использовать функцию `Playback()`, укажите имя файла в качестве аргумента. Например, воспроизведение `Playback(filename)`воспроизведёт звуковой файл с именем _filename.wav_, предполагая, что он находится в каталоге звуков по умолчанию. Обратите внимание, что вы можете включить полный путь к файлу, если хотите, например:

```text
Playback(/home/john/sounds/filename)
```

В предыдущем примере будет воспроизводиться _filename.wav_ из каталога _/home/john/sounds_. Это может быть проблематично из-за потенциальных проблем с правами доступа к файлам. Если вы планируете иметь много пользовательских звуков в своей системе, то вам, вероятно, понадобится выделенный каталог для них, и нужно будет проверить, чтобы Asterisk мог найти и воспроизвести файлы.

Вы также можете использовать относительные пути из каталога звуков Asterisk, как показано ниже:

```text
Playback(custom/filename)
```

В этом примере будет воспроизводиться _filename.wav_ из подкаталога _custom_ каталога звуков по умолчанию \(возможно _/var/lib/asterisk/sounds/en/custom/filename.wav_\). Если указанный каталог содержит более одного файла с этим именем, но с разными расширениями, Asterisk автоматически воспроизведёт лучший.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#asterisk-DP-Basics-FN-2)

Приложение `Hangup()` делает именно то, что следует из его названия: оно завершает активный канал. Вы должны использовать это приложение в конце контекста, когда хотите завершить текущий вызов, чтобы убедиться, что абоненты не продолжают выполнение диалплана таким образом, который вы, возможно, не ожидали. Приложение `Hangup()` не требует никаких аргументов, но вы можете передать код причины ISDN если захотите, например `Hangup(16)` и он будет переведен в сопоставимое сообщение SIP и отправлено на дальний конец.

По мере работы над книгой мы будем знакомить вас со многими другими приложениями Asterisk, но пока достаточно теории; давайте напишем диалплан!

### Базовый прототип диалплана

Таким образом, повторю, что форма всех диалпланов строится на основе этих четырех понятий: контекст, расширение, приоритет и приложение \(Рисунок6-3\).

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 6-3. &#x41F;&#x440;&#x43E;&#x442;&#x43E;&#x442;&#x438;&#x43F; &#x434;&#x438;&#x430;&#x43B;&#x43F;&#x43B;&#x430;&#x43D;&#x430;](.gitbook/assets/2.png)

## Простой диалплан

Ладно, хватит теории. Откройте файл _/etc/asterisk/extensions.conf_ в вашем любимом редакторе, и давайте посмотрим на ваш первый диалплан \(который был создан в [Главе 5](glava-05.md)\). Мы собираемся добавить к нему.

### Hello World

Как это обычно бывает во многих технологических книгах \(особенно в книгах по компьютерному программированию\), наш первый пример называется “Hello World.”

В первом приоритете нашего расширения мы отвечаем на вызов. Во втором мы проигрываем звуковой файл с именем _hello-world_, а в третьем вешаем трубку. Код, который нас интересует для этого примера выглядит так:

```text
exten => 200,1,Answer()
    same => n,Playback(hello-world)
    same => n,Hangup()
```

Если вы следовали в Главе 5, у вас уже будет настроен канал или два, а также пример диалплана, который содержит этот код. Если нет,то вам нужно расширение.файл conf в каталоге /etc / asterisk, содержащий следующий код:

```text
[general]
[globals]
[sets]
exten => 100,1,Dial(PJSIP/0000f30A0A01) ; Replace 0000f30A0A01 with your device name
exten => 101,1,Dial(PJSIP/SOFTPHONE_A)
exten => 102,1,Dial(PJSIP/0000f30B0B02)
exten => 103,1,Dial(PJSIP/SOFTPHONE_B)
exten => 200,1,Answer()
    same => n,Playback(hello-world)
    same => n,Hangup()
```

{% hint style="info" %}
**Tip**

If you don’t have any channels configured, now is the time to do so. There is real satisfaction that comes from passing your first call into an Asterisk dialplan on a system that you’ve built from scratch. People get this funny grin on their faces as they realize that they have just created a telephone system. This pleasure can be yours as well, so please, don’t go any further until you have made this little bit of dialplan work. If you have any problems, get back to [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig) and work through the examples there.
{% endhint %}

If you don’t have this dialplan code built yet, you’ll need to add it and reload the dialplan with this CLI command:

```text
$ sudo asterisk -rvvvvv # ('r' attaches to a daemonized Asterisk; 'v's are for verbosity)
*CLI> dialplan reload
```

or you can issue the command directly from the shell with:

```text
$ sudo asterisk -rx "dialplan reload" # ('rx' execute an Asterisk command and return)
```

Calling extension 200 from either of your configured phones[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408218280) should reward you with the friendly voice of Allison Smith saying “Hello, World.”

If it doesn’t work, check the Asterisk console for error messages, and make sure your channels are assigned to the sets context.

{% hint style="danger" %}
**Warning**

We do not recommend that you move forward in this book until you have verified the following:

1. Calls between extension 100 and 101 are working.
2. Calling extension 200 plays “Hello World.”
{% endhint %}

Even though this example is very short and simple, it emphasizes the core dialplan concepts of contexts, extensions, priorities, and applications. You now have the fundamental knowledge on which all dialplans are built.

{% hint style="success" %}
As you build out a dialplan, it will be helpful to have the Asterisk CLI open in a new window. You will be reloading the dialplan often, and while testing your call flow, you will want to see what is happening, as it happens. The Asterisk CLI is useful for both of those things.

```text
$ sudo asterisk -rvvvvv
*CLI> dialplan reload # this Asterisk CLI command reloads the dialplan
```

Best practice, then, would be to edit in one window, and to reload and debug in another.
{% endhint %}

## Building an Interactive Dialplan

The dialplan we just built was static; it will always perform the same actions on every call. Many dialplans will also need logic to perform different actions based on input from the user, so let’s take a look at that now.

### The Goto\(\), Background\(\), and WaitExten\(\) Applications

As its name implies, the Goto\(\) application is used to send a call to another part of the dialplan. Goto\(\) requires us to pass the destination context, extension, and priority as arguments, like this:

```text
 same => n,Goto(context,extension,priority)
```

We’re going to create a new context called TestMenu, and create an extension in our sets context that will pass calls to that context using Goto\(\):

```text
exten => 200,1,Answer()
 same => n,Playback(hello-world)
 same => n,Hangup()
exten => 201,1,Goto(TestMenu,start,1) ; add this to the end of the
 ; [sets] context
[TestMenu]
exten => start,1,Answer()
```

Now, whenever a device enters the \[sets\] context and dials 201, the call will be passed to the start extension in the TestMenu context \(which currently won’t do anything interesting because we still have more code to write\).

{% hint style="info" %}
**Note**

We used the extension start in this example, but we could have used anything we wanted as an extension name, either numeric or alpha. We prefer to use alpha characters for extensions that are not directly dialable, as this makes the dialplan easier to read. Point being, we could have named our target extension 123 or xyz321, or 99luftballons, or whatever we wanted instead of start. The word start doesn’t mean anything special to the dialplan; it’s simply the name of an extension.
{% endhint %}

One of the more useful applications in an interactive Asterisk dialplan is the Background\(\)[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408187512) application. Like Playback\(\), it plays a recorded sound file. Unlike Playback\(\), however, when the caller presses a key \(or series of keys\) on their telephone keypad, it interrupts the playback and passes the call to the extension that corresponds with the pressed digit\(s\). If a caller presses 5, for example, Asterisk will stop playing the sound prompt and send control of the call to the first priority of extension 5 \(assuming there is an extension 5 to send the call to\).

The most common use of the Background\(\) application is to create basic voice menus \(often called auto attendants, IVRs,[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408181448) or phone trees\). Many companies use voice menus to direct callers to the proper extensions, thus relieving their receptionists from having to answer every single call.

Background\(\) has the same syntax as Playback\(\):

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
```

If you want Asterisk to wait for input from the caller after the sound prompt has finished playing, you can use WaitExten\(\). The WaitExten\(\) application waits for the caller to enter DTMF digits and is used directly following the Background\(\) application, like this:

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten()
```

If you’d like the WaitExten\(\) application to wait a specific number of seconds for a response \(instead of using the default timeout\),[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408171000) simply pass the number of seconds as the first argument to WaitExten\(\), like this:

```text
 same => n,WaitExten(5) ; We always pass a time argument to WaitExten()
```

Both Background\(\) and WaitExten\(\) allow the caller to enter DTMF digits. Asterisk then attempts to find an extension in the current context that matches the digits that the caller entered. If Asterisk finds a match, it will send the call to that extension. Let’s demonstrate by adding a few lines to our dialplan example:

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Playback(digits/1)
exten => 2,1,Playback(digits/2)
```

After making these changes, save and reload your dialplan:

```text
*CLI> dialplan reload
```

If you call into extension 201, you should hear a sound prompt that says, “Enter the extension of the person you are trying to reach.” The system will then wait 5 seconds for you to enter a digit. If the digit you press is either 1 or 2, Asterisk will match the relevant extension, and read that digit back to you. Since we didn’t provide any further instructions, your call will then end. You’ll also find that if you enter a different digit \(such as 3\), the dialplan will be unable to proceed.

Let’s embellish things a little. We’re going to use the Goto\(\) application to have the dialplan repeat the greeting after playing back the number:

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Playback(digits/1)
 same => n,Goto(TestMenu,start,1)
exten => 2,1,Playback(digits/2)
 same => n,Goto(TestMenu,start,1)
```

These new lines will send control of the call back to the start extension after playing back the selected number.

{% hint style="info" %}
**Tip**

If you look up the details of the Goto\(\) application, you’ll find that you can actually pass either one, two, or three arguments to the application. If you pass a single argument, Asterisk will assume it’s the destination priority in the current extension. If you pass two arguments, Asterisk will treat them as the extension and the priority to go to in the current context.

In this example, we’ve passed all three arguments for the sake of clarity, but passing just the extension and priority would have had the same effect, since the destination context is the same as the source context.
{% endhint %}

### Handling Invalid Entries and Timeouts

We need an extension for invalid entries. In Asterisk, when a context receives a request for an extension that is not valid within that context \(e.g., pressing 9 in the preceding example\), the call is sent to the i extension. We also need an extension to handle situations when the caller doesn’t give input in time \(the default timeout is 10 seconds\). Calls will be sent to the t extension if the caller takes too long to press a digit after WaitExten\(\) has been called. Here is what our dialplan will look like after we’ve added these two extensions:

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Playback(digits/1)
 same => n,Goto(TestMenu,start,1)
exten => 2,1,Playback(digits/2)
 same => n,Goto(TestMenu,start,1)
exten => i,1,Playback(pbx-invalid)
 same => n,Goto(TestMenu,start,1)
exten => t,1,Playback(please-try-again)
 same => n,Goto(TestMenu,start,1)
```

Using the i[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408142552) and t extensions makes our menu a little more robust and user-friendly. That being said, it is still quite limited, because outside callers still have no way of connecting to a live person. To do that, we’ll need to learn about the Dial\(\) application.

### Using the Dial\(\) Application

One of Asterisk’s most valuable features is its ability to connect different callers to each other. While Asterisk currently is used mostly for SIP connections, it supports a wide variety of channel types \(from Analog to SS7, and various old VoIP protocols such as MGCP and SCCP\). Asterisk takes much of the hard work out of connecting and translating between disparate networks. All you have to do is learn how to use the Dial\(\) application.

The syntax of the Dial\(\) application is more complex than that of the other applications we’ve used so far, but it’s also where much of the magic of Asterisk happens. Dial\(\) takes up to four arguments, which we’ll look at next.

The syntax of Dial\(\) looks like this:

```text
Dial(Technology/Resource[&Technology2/Resource2[&...]][,timeout[,options[,URL]]])
```

Put simply, you tell Dial\(\) what channel[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408129896) you want to send the call out to, and set a few options to tweak the behavior. The use of Dial\(\) can get complex, but at its most basic, it’s that simple.

#### Argument 1: destination

The first argument is the destination you’re attempting to call, which \(in its simplest form\) is made up of a technology \(or transport\) across which to make the call, a forward slash, and the address of the remote endpoint or resource.

{% hint style="info" %}
**Note**

These days, you’re most likely to be using PJSIP as your channel type, but in the not-too-distant past, common technology types also included DAHDI \(for analog and T1/E1/J1 channels\), the old SIP channel \(prior to PJSIP\), and IAX2.[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408125064) If you’re looking at an older dialplan, you may see some of these other protocols represented. Going forward, only PJSIP and DAHDI are recommended and maintained.
{% endhint %}

Let’s assume that we want to call one of our PJSIP channels named SOFTPHONE\_B. The technology is PJSIP, and the resource \(or channel\) identifier is SOFTPHONE\_B. Similarly, a call to a DAHDI device \(defined in chan\_dahdi.conf\) might have a destination of DAHDI/14169671111. If we wanted Asterisk to ring the PJSIP/SOFTPHONE\_B channel when extension 103 is reached in the dialplan, we’d add the following extension:

```text
exten => 101,1,Dial(PJSIP/SOFTPHONE_A)
exten => 103,1,Dial(PJSIP/SOFTPHONE_B)
exten => 200,1,Answer()
```

We can also dial multiple channels at the same time, by concatenating the destinations with an ampersand \(&\), like this:

```text
exten => 101,1,Dial(PJSIP/SOFTPHONE_A)
exten => 103,1,Dial(PJSIP/SOFTPHONE_B)
exten => 110,1,Dial(PJSIP/0000f30A0A01&PJSIP/SOFTPHONE_A&PJSIP/SOFTPHONE_B)
exten => 200,1,Answer()
```

The Dial\(\) application will ring all of the specified destinations simultaneously, and bridge the inbound call with whichever destination channel answers first \(the other channels will immediately stop ringing\). If the Dial\(\) application can’t contact any of the destinations, Asterisk will set a variable called DIALSTATUS with the reason that it couldn’t dial the destinations, and continue with the next priority in the extension.[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-3)

The Dial\(\) application also allows you to connect to a remote VoIP endpoint not previously defined in one of the channel configuration files. The full syntax is:

```text
Dial(technology/user[:password]@remote_host[:port][/remote_extension])
```

The full syntax for the Dial\(\) application is slightly different for DAHDI channels:

```text
Dial(DAHDI/[gGrR]channel_or_group[/remote_extension])
```

For example, here is how you would dial 1-800-555-1212 on DAHDI channel number 4:[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408102536)

```text
exten => 501,1,Dial(DAHDI/4/18005551212)
```

#### Argument 2: timeout

The second argument to the Dial\(\) application is a timeout, specified in seconds. If a timeout is given, Dial\(\) will attempt to call the specified destination\(s\) for that number of seconds before giving up and moving on to the next priority in the extension. If no timeout is specified, Dial\(\) will continue to dial the called channel\(s\) until someone answers or the caller hangs up. Let’s add a timeout of 10 seconds to our extension:

```text
exten => 101,1,Dial(PJSIP/SOFTPHONE_A)
exten => 102,1,Dial(PJSIP/0000f30B0B02,10)
exten => 103,1,Dial(PJSIP/SOFTPHONE_B)
```

If the call is answered before the timeout, the channels are bridged and the dialplan is done. If the destination simply does not answer, is busy, or is otherwise unavailable, Asterisk will set a variable called DIALSTATUS and then continue on with the next priority in the extension.

Let’s put what we’ve learned so far into another example:

```text
exten => 102,1,Dial(PJSIP/0000f30B0B02,10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
```

As you can see, this example will play the vm-nobodyavail.gsm sound file if the call goes unanswered \(and then hang up\). Note that this doesn’t actually provide voicemail; we’re just playing a prompt, which could have been any valid prompt. We’ll cover sending calls to voicemail later.

#### Argument 3: option

The third argument to Dial\(\) is an option string. It may contain one or more characters that modify the behavior of the Dial\(\) application. While the list of possible options is too long to cover here, one of the most popular is the m option. If you place the letter m as the third argument, the calling party will hear hold music instead of ringing while the destination channel is being called \(assuming, of course, that music on hold has been configured correctly\). To add the m option to our last example, we simply change the first line:

```text
exten => 102,1,Dial(PJSIP/0000f30B0B02,10,m)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
```

#### Argument 4: URI

The fourth and final argument to the Dial\(\) application is a URI. If the destination channel supports receiving a URI at the time of the call, the specified URI will be sent \(for example, if you have an IP telephone that supports receiving a URI, it will appear on the phone’s display; likewise, if you’re using a softphone, the URI might pop up on your computer screen\). This argument is very rarely used.

#### Updating the dialplan

Let’s modify extensions 1 and 2 in our menu to use the Dial\(\) application, and add extensions 3 and 4 just for good measure:

```text
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Dial(PJSIP/0000f30A0A01,10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 2,1,Dial(PJSIP/0000f30B0B02,10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 3,1,Dial(PJSIP/SOFTPHONE_A,10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 4,1,Dial(PJSIP/SOFTPHONE_B,10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => i,1,Playback(pbx-invalid)
 same => n,Goto(TestMenu,start,1)
exten => t,1,Playback(vm-goodbye)
 same => n,Hangup()
```

#### Blank arguments

Note that the second, third, and fourth arguments may be left blank; only the first argument is required. For example, if you want to specify an option but not a timeout, simply leave the timeout argument blank, like this:

```text
exten => 4,1,Dial(SIP/SOFTPHONE_B,,m)
```

### Using Variables

If you have programming experience, you already understand what a variable is. If not, we’ll briefly explain what variables are and how they are used. Any dialplan work beyond the very simple examples just given will greatly benefit from the use of variables. They are one of the useful features of a customizable dialplan that you will not find in a typical proprietary PBX.

A variable is a named container that can hold a value. Think of it like a post office box. The advantage of a variable is that its contents may change, but its name does not, which means you can write code that references the variable name and not worry about what the value will be. It is almost impossible to do any sort of useful programming without variables.

There are two ways to reference a variable. To reference the variable’s name, simply type the name of the variable. If, on the other hand, you want to reference the value of the variable, you must type a dollar sign, an opening curly brace, the name of the variable, and a closing curly brace. So, to use the post office box analogy, you refer to the box itself by simply using its name, and you refer to the contents with the use of the ${} wrapper. A variable named MyVar is referred to as MyVar, and its contents are accessed with ${MyVar}. Here’s how we might use a variable inside the Dial\(\) application:[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408063256)

```text
exten => 203,1,Noop(say some digits)
 same => n,Answer()
 same => n,Set(SomeDigits=123)
 same => n,SayDigits(${SomeDigits})
 same => n,Wait(.25)
 same => n,Set(SomeDigits=543)
 same => n,SayDigits(${SomeDigits})
```

In our dialplan, whenever we refer to ${SomeDigits}, Asterisk will automatically replace it with whatever value has been assigned to the variable named SomeDigits.

{% hint style="info" %}
**Tip**

Note that variable names are case-sensitive. A variable named SOMEDIGITS is different from a variable named SomeDigits. You should also be aware that any variables set by Asterisk will be uppercase. Some variables, such as CHANNEL and EXTEN, are reserved by Asterisk. You should not attempt to set these variables. It is popular to write global variables in uppercase and channel variables in Pascal/Camel case, but it is not strictly required.
{% endhint %}

There are three types of variables we can use in our dialplan: global variables, channel variables, and environment variables. Let’s take a moment to look at each type.

#### Global variables

As their name implies, global variables are visible to all channels at all times. Global variables are useful in that they can be used anywhere within a dialplan to increase readability and manageability. Suppose for a moment that you had a large dialplan and several hundred references to the PJSIP/0000f30A0A01 channel. Now imagine you replaced the phone with a different unit \(perhaps a different MAC address\), and had to go through your dialplan and change all of those references to PJSIP/0000f30A0A01. Not pretty.

On the other hand, if you had defined a global variable that contained the value PJSIP/0000f30A0A01 at the beginning of your dialplan and then referenced that instead, you would have to change only one line of code to affect all places in the dialplan where that channel was used.

Global variables should be declared in the \[globals\] context at the beginning of the extensions.conf file. As an example, we will create a few global variables that store the channel identifiers of our devices. These variables are set at the time Asterisk parses the dialplan:

```text
[globals]
UserA_DeskPhone=PJSIP/0000f30A0A01
UserA_SoftPhone=PJSIP/SOFTPHONE_A
UserB_DeskPhone=PJSIP/0000f30B0B02
UserB_SoftPhone=PJSIP/SOFTPHONE_B
```

We’ll come back to these later.

#### Channel variables

A channel variable is a variable that is associated only with a particular call. Unlike global variables, channel variables are defined only for the duration of the current call and are available only to the channels participating in that call.

There are many predefined channel variables available for use within the dialplan, which are explained in the [Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Channel+Variables). You define a channel variable with extension 203 and the Set\(\) application:

```text
exten => 203,1,Noop(say some digits)
 same => n,Set(SomeDigits=123)
 same => n,SayDigits(${SomeDigits})
 same => n,Wait(.25)
 same => n,Set(SomeDigits=543)
 same => n,SayDigits(${SomeDigits})
```

You’re going to be seeing a lot more channel variables. Read on.

#### Environment variables

Environment variables are a way of accessing Unix environment variables from within Asterisk. These are referenced using the ENV\(\) dialplan function.[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408032008) The syntax looks like ${ENV\(var\)}, where var is the Unix environment variable you wish to reference. Environment variables aren’t commonly used in Asterisk dialplans, but they are available should you need them.

#### Adding variables to our dialplan

Now that we’ve learned about variables, let’s put them to work in our dialplan. We’re going to add three global variables that will associate a variable name to a channel name:

```text
[general]
[globals]
UserA_DeskPhone=PJSIP/0000f30A0A01
UserA_SoftPhone=PJSIP/SOFTPHONE_A
UserB_DeskPhone=PJSIP/0000f30B0B02
UserB_SoftPhone=PJSIP/SOFTPHONE_B
[sets]
exten => 100,1,Dial(${UserA_DeskPhone})
exten => 101,1,Dial(${UserA_SoftPhone})
exten => 102,1,Dial(${UserB_DeskPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 103,1,Dial(${UserB_SoftPhone})
exten => 110,1,Dial(${UserA_DeskPhone}&${UserA_SoftPhone}&${UserB_SoftPhone})
exten => 200,1,Answer()
Let’s update the test menu as well:
[TestMenu]
exten => start,1,Answer()
 same => n,Background(enter-ext-of-person)
 same => n,WaitExten(5)
exten => 1,1,Dial(${UserA_DeskPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 2,1,Dial(${UserA_SoftPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 3,1,Dial(${UserB_DeskPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => 4,1,Dial(${UserB_SoftPhone},10)
 same => n,Playback(vm-nobodyavail)
 same => n,Hangup()
exten => i,1,Playback(pbx-invalid)
```

It rarely makes sense to hardcode data in a dialplan. It’s almost always better to use a variable.

Make sure you test this out to ensure you don’t have any typos, and also to see what it looks like when executed on the Asterisk CLI:

```text
# asterisk -rvvvvvv
*CLI> dialplan reload
 -- Executing [201@sets:1] Goto("PJSIP/0000f30A0A01", "TestMenu,start,1")
 -- Goto (TestMenu,start,1)
 -- Exec [start@TestMenu:1] Answer("PJSIP/0000f30A0A01", "")
 -- Exec [start@TestMenu:2] BackGround("PJSIP/0000f30A0A01", "enter-ext-of-person")
 -- <PJSIP/0000f30A0A01> Playing 'enter-ext-of-person.slin' (language 'en')
 -- Exec [1@TestMenu:1] Dial("PJSIP/0000f30A0A01", "PJSIP/0000f30A0A01,10")
 -- Called PJSIP/0000f30A0A01
 -- PJSIP/0000f30A0A01-00000011 is ringing
 == Spawn extension (TestMenu, 1, 1) exited non-zero on 'PJSIP/0000f30A0A01'
```

#### Variable concatenation

To concatenate variables, simply place them together, like this:

```text
exten => 204,1,Answer()
 same => n,Answer()
 same => n,Set(ONETWO=12)
 same => n,Set(THREEFOUR=34)
 same => n,SayDigits(${ONETWO}${THREEFOUR}) ; easy peasy
 same => n,Wait(0.2)
 same => n,Set(NOTFIVE=${THREEFOUR}${ONETWO}) ; peasy easy
 same => n,SayNumber(${NOTFIVE}) ; see what we did here?
 same => n,Wait(0.2)
 same => n,SayDigits(2${ONETWO}3) ; you can concatenate literals and variables
```

#### Inheriting channel variables

Channel variables are always associated with the original channel that set them, and are no longer available once the channel is transferred.

In order to allow channel variables to follow the channel as it is transferred around the system, you must employ channel variable inheritance. There are two modifiers that can allow the channel variable to follow the channel: single underscore and double underscore.

The single underscore \(\_\) causes the channel variable to be inherited by the channel for a single transfer, after which it is no longer available for additional transfers. If you use a double underscore \(\_\_\), the channel variable will be inherited throughout the life of that channel.

Setting channel variables for inheritance simply requires you to prefix the channel name with a single or double underscore. The channel variables are then referenced exactly the same as they would be normally.

Here’s an example of setting a channel variable for single transfer inheritance:

```text
exten => example,1,Set(_MyVariable=thisValue)
```

Here’s an example of setting a channel variable for infinite transfer inheritance:

```text
exten => example,1,Set(__MyVariable=thisValue)
```

When you wish to read the value of the channel variable, you do not use the underscore\(s\):

```text
exten => example,1,Verbose(1,Value of MyVariable is: ${MyVariable})
```

### Pattern Matching

If we want to be able to allow people to dial through Asterisk and have Asterisk connect them to outside resources, we need a way to match on any possible phone number that the caller might dial. For situations like this, Asterisk offers pattern matching. Pattern matching allows you to create one extension in your dialplan that matches many different numbers. This is enormously useful.

#### Pattern-matching syntax

When we are using pattern matching, certain letters and symbols represent what we are trying to match. Patterns always start with an underscore \(\_\). This tells Asterisk that we’re matching on a pattern, and not on an explicit extension name.

**Warning**

If you forget the underscore at the beginning of your pattern, Asterisk will think it’s just a named extension and won’t do any pattern matching. This is one of the most common mistakes people make when starting to learn Asterisk.

After the underscore, you can use one or more of the following characters:

X

Matches any single digit from 0 to 9.

Z

Matches any single digit from 1 to 9.

N

Matches any single digit from 2 to 9.

**Note**

Another common mistake is to try to use the letters X, Z, and N literally in a pattern match; to do that, wrap them in square brackets \(case-insensitive\), such as \_ale\[X\]\[Z\]a\[N\]der.

\[15-7\]

Matches a single character from the range of digits specified. In this case, the pattern matches a single 1, as well as any number in the range 5, 6, 7.

. \(period\)

Wildcard match; matches one or more characters, no matter what they are.

**Warning**

If you’re not careful, wildcard matches can make your dialplans do things you’re not expecting \(like matching built-in extensions such as i or h\). You should use the wildcard match in a pattern only after you’ve matched as many other digits as possible. For example, the following pattern match should probably never be used:

\_.

In fact, Asterisk will warn you if you try to use it. Instead, if you really need a catchall pattern match, use this one to match all strings that start with a digit followed by one or more characters \(see ! if you want to be able to match on zero or more characters\):

\_X.

Or this one, to match any alphanumeric string:

\_\[0-9a-zA-Z\].

! \(bang\)

Wildcard match; matches zero or more characters, no matter what they are.

To use pattern matching in your dialplan, simply put the pattern in the place of the extension name \(or number\):

exten =&gt; \_4XX,1,Noop\(User Dialed ${EXTEN}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

 same =&gt; n,Hangup\(\)

In this example, the pattern matches any three-digit extension from 400 through 499.[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178407965128)

One other important thing to know about pattern matching is that if Asterisk finds more than one pattern that matches the dialed extension, it will use the most specific one \(going from left to right\). Say you had defined the following two patterns, and a caller dialed 555-1212:

exten =&gt; \_555XXXX,1,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

exten =&gt; \_55512XX,1,Answer\(\)

 same =&gt; n,Playback\(tt-monkeys\)

In this case the second extension would be selected, because it is more specific. Load this in and make calls to 5550000, 5550123, 5551212, 5551200, 5551300, 5551299, and so forth to get a feel for how this works. Play around with different pattern matches. For example, what would pattern \_555NNNN match? What would pattern \_\[0-9\]. match?

#### North American Numbering Plan—pattern-matching examples

This pattern matches any seven-digit number, as long as the first digit is 2 or higher:

\_NXXXXXX

The preceding pattern would be compatible with any North American Numbering Plan local seven-digit number.

In areas with 10-digit dialing, that pattern would look like this:

\_NXXNXXXXXX

Note that neither of these two patterns would handle long-distance calls. We’ll cover those shortly.

**The NANP and Toll Fraud**

The North American Numbering Plan \(NANP\) is a shared telephone numbering scheme used by 19 countries in North America and the Caribbean. All of these countries share country code 1.

In the United States and Canada, there is sufficient competition that you can place a long-distance call to most numbers in country code 1 and expect to pay a reasonable toll. However, many people don’t realize that 17 other countries, many of which have very different telecom regulations, [share the NANP](http://www.nanpa.com/). Some of these places are quite expensive to call.

One popular scam using the NANP tries to trick naïve North Americans into calling expensive per-minute toll numbers in a Caribbean country; the callers believe that since they dialed 1-NPA-NXX-XXXX to reach the number, they’ll be paying their standard national long-distance rate for the call. Since the country in question may have regulations that allow for this form of extortion, the caller is ultimately held responsible for the call charges.

It may be prudent to block calls to area codes to NANP countries outside the US and Canada until you’ve had a chance to review your toll rates to those countries. Wikipedia has [a good reference](http://bit.ly/2Ztku7l) for the basics of what you need to know about NANP, including what NPAs \(area codes\) belong to what country.

Let’s try another:

\_1NXXNXXXXXX

This one will match the number 1, followed by an area code between 200 and 999, then any seven-digit number that does not start with 0 or 1. In the NANP calling area, you would use this pattern to match any long-distance number.[19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-8)

And finally this one:

\_011.

Note the period on the end. This pattern matches any number that starts with 011 and has at least one more digit. In the NANP, this indicates an international phone number. \(We’ll be using these patterns in the next section to add outbound dialing capabilities to our dialplan.\)

#### Common global pattern matches

Outside of North America, there is wide variance in how numbering is handled; however, some patterns are common. Here are a few simple examples:

```text
; UK, Germany, Italy, China, etc.
exten => _00X.,1,noop() ; international dialing code
exten => _0X.,1,noop() ; national dialing prefix
exten => 112,1,Noop(--==[ Emergency call ]==--)
; Australia
exten => _0011X.,1,noop() ; international dialing code
exten => _0X.,1,noop() ; national dialing prefix
; Dutch Caribbean (Saba)
exten => _00X.,1,noop() ; international
exten => _416XXXX,1,noop() ; local (on-island)
exten => _0[37]XXXXXX,1,noop() ; call to country code 599 off-island (not Curacao)
exten => _09XXXXXXX,1,Noop() ; call to country code 599 off-island (Curacao)
```

You will need to understand the dialing plan of your region in order to produce a useful pattern match.

#### Using the ${EXTEN} channel variable

So what happens if you want to use pattern matching but need to know which digits were actually dialed? Enter the ${EXTEN} channel variable. Whenever you dial an extension, Asterisk sets the ${EXTEN} channel variable to the digits that were received. We used the application SayDigits\(\) to demonstrate this.

```text
exten => _4XX,1,Noop(User Dialed ${EXTEN})
 same => n,Answer()
 same => n,SayDigits(${EXTEN})
 same => n,Hangup()
exten => _555XXXX,1,Answer()
 same => n,SayDigits(${EXTEN})
```

In these examples, the SayDigits\(\) application read back to you the extension you dialed.

Often, it’s useful to manipulate the ${EXTEN} by stripping a certain number of digits off the front of the extension. This is accomplished by using the syntax ${EXTEN:x}, where x is where you want the returned string to start, from left to right. For example, if the value of ${EXTEN} is 95551212, ${EXTEN:1} equals 5551212. Let’s try another example:

```text
exten => _XXX,1,Answer()
 same => n,SayDigits(${EXTEN:1})
```

In this example, the SayDigits\(\) application would start at the second digit, and thus read back only the last two digits of the dialed extension.

**More Advanced Digit Manipulation**

The ${EXTEN} variable properly has the syntax ${EXTEN:x:y}, where x is the starting position and y is the number of digits to return. Given the following dial string:

94169671111

we can extract the following digit strings using the ${EXTEN:x:y} construct:

* ${EXTEN:1:3} would contain 416
* ${EXTEN:4:7} would contain 9671111
* ${EXTEN:-4:4} would start four digits from the end and return four digits, giving us 1111
* ${EXTEN:2:-4} would start two digits in and exclude the last four digits, giving us 16967
* ${EXTEN:-6:-4} would start six digits from the end and exclude the last four digits, giving us 67
* ${EXTEN:1} would give us everything after the first digit, or 4169671111 \(if the number of digits to return is left blank, it will return the entire remaining string\)

This is a very powerful construct, but most of these variations are not very common in normal use. For the most part, you will be using ${EXTEN} \(or perhaps ${EXTEN:1} if you need to strip off an external access code, such as a prepended 9\).

### Includes

Asterisk has an important feature that allows extensions from one context to be available from within another context. This is accomplished through use of the include directive, which allows us to control access to different sections of the dialplan.

The include statement takes the following form, where context is the name of the remote context we want to include in the current context:

```text
include => context
```

Including one context within another context allows extensions within the included context to be dialable.

When we include other contexts within our current context, we have to be mindful of the order in which we are including them. Asterisk will first try to match the dialed extension in the current context. If unsuccessful, it will then try the first included context \(including any contexts included in that context\), and then continue to the other included contexts in the order in which they were included.

We will discuss the include directive more in [Chapter 7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22asterisk-OutsideConn).

## Conclusion

And there you have it—a basic but functional dialplan. There is still much we have not covered, but you’ve got all of the fundamentals. In the following chapters, we’ll continue to build on this foundation.

If parts of this dialplan don’t make sense, you may want to go back and reread a section or two before continuing on to the next chapter. It’s imperative that you understand these principles and how to apply them, as the next chapters build on this information.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-1-marker) Please note that the space is conspicuously absent from the list of allowed characters. Don’t use spaces in your context names—you won’t like the result!

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408408152-marker) The default context used to be a popular way to whip up simple configurations, but this proved to be somewhat problematic for security. Best practice these days is to avoid all use of it.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408333272-marker) Asterisk permits simple arithmetic within the priority, such as n+200, and the priority s \(for same\), but their usage is somewhat deprecated due to the existence of priority labels. Please note that extension s and priority s are two distinct concepts.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408295288-marker) OK, so feeding the cat isn’t a common use for a telephone system, but through Asterisk, such things are not impossible. Doc Brown would’ve loved this thing.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408274728-marker) There is another application called Background\(\) that is very similar to Playback\(\), except that it does allow input from the caller. You can read more about this application in Chapters [14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA) and [16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22asterisk-IVR).

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-2-marker) Asterisk selects the best file based on translation cost—that is, it selects the file that is the least CPU-intensive to convert to its native audio format. When you start Asterisk, it calculates the translation costs between the different audio formats \(they often vary from system to system\). You can see these translation costs by typing core show translation at the Asterisk CLI. The numbers shown represent how many microseconds it takes Asterisk to transcode one second of audio.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408218280-marker) If you haven’t configured two phones yet, please consider heading back to [Chapter 5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch05.html%22%20/l%20%22asterisk-DeviceConfig) and getting a couple of phones set up so you can play with them. You can get away with only one phone for testing, but really two is ideal. There are lots of free softphones available, and some of them are rather good.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408187512-marker) It should be noted that some people expect that Background\(\), due to its name, will continue onward through the next steps in the dialplan while the sound is being played. In reality, its name refers to the fact that it is playing a sound in the background, while waiting for DTMF in the foreground.

[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408181448-marker) More information about auto attendants and IVR can be found in [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA).

[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408171000-marker) See the dialplan function TIMEOUT\(\) for information on how to change the default timeouts. See [Chapter 10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-DP-Deeper) for information on what dialplan functions are.

[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408142552-marker) The i extension is for catching invalid entries supplied to a dialplan application such as Background\(\). It is not used for matching on invalidly dialed extensions or nonmatching pattern matches.

[12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408129896-marker) Or channels, if you want to ring more than one at a time.

[13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408125064-marker) IAX2 \(pronounced “EEKS”\), is the Inter Asterisk Exchange protocol \(v2\). In the early days of Asterisk it was popular for trunking, as it greatly reduced signaling overhead on busy circuits. Bandwidth has become far less expensive, and SIP protocol has become nearly ubiquitous. The IAX2 protocol is no longer actively maintained, but it still retains some popularity for its ability to traverse firewalls, and a few carriers might still support it. However, its use is deprecated, and in fact discouraged.

[14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-3-marker) We’ll cover variables in the section [“Using Variables”](6.%20Dialplan%20Basics%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-DP-Basics-SECT-3.5). In future chapters we’ll discuss how to have your dialplan make decisions based on the value of DIALSTATUS.

[15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408102536-marker) Bear in mind that this assumes that this channel connects to something that knows how to reach external numbers.

[16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408063256-marker) Specifically, what we are setting here is a channel variable.

[17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178408032008-marker) We’ll get into dialplan functions later. Don’t worry too much about environment variables right now. They are not important to understanding the dialplan.

[18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22idm46178407965128-marker) We’ve used the EXTEN channel variable without introducing it. Read on, as it will be covered later in this chapter.

[19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics-FN-8-marker) If you grew up in North America, you may believe that the 1 you dial before a long-distance call is “the long-distance code.” This is not completely correct. The number 1 is also the international country code for NANP. Keep this in mind if you send your phone number to someone in another country. The recipient may not know your country code, and thus be unable to call you with just your area code and phone number. Your full phone number with country code is +1 NPA NXX XXXX \(where NPA is your area code\)—for example, +1 416 555 1212. This is also known as E.164 format \([Wikipedia](http://bit.ly/2ImniNO) can tell you all about E.164\).

