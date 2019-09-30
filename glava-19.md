---
description: Asterisk REST Interface
---

# Глава 19

> Люди, которые думаю, будто все знают, раздражают нас, людей, которые действительно все знают.
>
> -- Айзек Азимов

Интерфейс Asterisk REST \(ARI\) был создан для устранения ограничений, присущих разработке внешних или расширенных функций вне Asterisk. В то время как AGI позволяет запускать внешние приложения, а AMI позволяет осуществлять наблюдение и контроль выполняемых вызовов, любая попытка интегрировать их в полное внешнее приложение быстро становится сложной и запутанной. ARI позволяет разработчикам создавать автономное и полное приложение, используя Asterisk в качестве базового движка.

На момент написания этой статьи ARI требует очень простого диалплана для запуска приложения `Stasis()`, которое затем передает канал ARI. К тому времени, когда вы читаете это, очень вероятно, что это требование изменилось, поскольку сообщество разработчиков Asterisk активно работает над тем, чтобы позволить ARI появляться без какого-либо диалплана в середине.

Использование внешнего интерфейса, такого как ARI, для управления Asterisk, не обязательно облегчит вашу жизнь. Навыки, необходимые для реализации и устранения неполадок приложений этого типа, требуют комплексного набора навыков не только на выбранном языке, но и в области системного администрирования Linux, администрирования Asterisk, устранения неполадок в сети и основных концепций телефонии. Для опытного разработчика ARI может дать необходимую мощь в приложениях, но для тех, кто учится, мы рекомендуем изучить диалплан прежде чем погружаться во внешние среды разработки. Диалплан является своеобразным, но он также полностью интегрирован, высокопроизводителен и относительно прост в освоении.

Сказав это, давайте разберемся с ARI.

## ARI быстрый старт

В этом разделе приведен простой рабочий пример ARI. Позже в этой главе мы рассмотрим все более подробно.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178403407976)

{% hint style="danger" %}
#### Предупреждение

В этом разделе быстрого старта мы будем использовать очень простой уровень доступа HTTP. Вы должны быть очень осторожны при вводе такого рода конфигурации в продакшен. Если, например, вы собираетесь запустить приложение на отдельном компьютере и подключить его к Asterisk через сокет, то потребуется более безопасное соединение. То, что мы делаем в этом разделе, сродни парусному клубу, использующему шлюпки для обучения; полезно в качестве начинания, но глупо и опасно выходить в море на таком судне.
{% endhint %}

### Базовая конфигурация Asterisk

У вас уже должен быть запущен веб-сервер Asterisk, поэтому вам просто нужно проверить, что ваш файл _/etc/asterisk/http.conf_ выглядит следующим образом:

```text
[general]
enabled = yes
bindaddr = 127.0.0.1
```

Далее нужен простой файл _/etc/asterisk/ari.conf_:

```text
[general]
enabled = yes
pretty = yes
[asterisk]
type = user
read_only = no
password = чтобывыниделалинеиспользуйтеэтотпароль
```

Хорошо, давайте загрузим модуль `ari` сейчас:

```text
$ sudo asterisk -rx 'module load res_ari.so'
Loaded res_ari.so => (Asterisk RESTful Interface)
```

Затем в файл _/etc/asterisk/extensions.conf_ необходимо добавить расширение для запуска приложения диалплана `Stasis()`:[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396697256)

```text
exten => 242,1,Noop()
 same => n,Stasis(zarniwoop)
 same => n,Hangup()
```

Перезагрузите ваш диалплан с помощью:

```text
$ sudo asterisk -rx 'dialplan reload'
Dialplan reloaded.
```

На этом этапе может быть стоит просто перезагрузить Asterisk:

```text
$ sudo service asterisk restart
```

Осталось всего несколько шагов, и вы готовы протестировать свою среду ARI.

### Тестирование вашей среды ARI

Поскольку ARI зависит от WebSockets, нам понадобится инструмент, позволяющий тестировать из командной строки. Node.JS package manager \(npm\) позволит нам найти и установить инструмент сканирования, который мы будем использовать для наших тестов.

```text
$ sudo yum -y install npm
$ sudo npm install -g wscat
/usr/bin/wscat -> /usr/lib/node_modules/wscat/bin/wscat
/usr/lib
+-- wscat@2.2.1
  +-- commander@2.15.1
  +-- read@1.0.7
  ¦ +-- mute-stream@0.0.8
  +-- ws@5.2.2
    +-- async-limiter@1.0.0
```

Теперь давайте зажжем его и посмотрим, что получим!

```text
$ wscat -c "ws://localhost:8088/ari/events?api_key= \
 asterisk:чтобывыниделалинеиспользуйтеэтотпароль&app=zarniwoop"

connected (press CTRL+C to quit)
>
```

Пока все идет хорошо. Давайте сделаем звонок в наше приложение `Stasis()` и посмотрим что произойдет.

Откройте новое окно SSH \(оставьте предыдущее как есть, чтобы видеть что происходит в сеансе `wscat`\). Подключитесь к CLI Asterisk в этом новом сеансе оболочки:

```text
$ sudo asterisk -rvvvv
```

Используя один из ваших лабораторных телефонов, позвоните на номер 242.

В Asterisk CLI, вы должны увидеть это:

```text
*CLI>
 == Setting global variable 'SIPDOMAIN' to '172.29.1.57'
 -- Executing [242@sets:1] NoOp("PJSIP/SOFTPHONE_A-00000001", "") in new stack
 -- Executing [242@sets:2] Stasis("PJSIP/SOFTPHONE_A-00000001", "zarniwoop") in new stack
```

И в сеансе `wscat` вы должны увидеть это:

```text
>
< {
  "type": "StasisStart",
  "timestamp": "2019-01-27T21:43:43.720-0500",
  "args": [],
  "channel": {
    "id": "1548643423.2",
    "name": "PJSIP/SOFTPHONE_A-00000002",
    "state": "Ring",
    "caller": {
      "name": "101",
      "number": "SOFTPHONE_A"
    },
    "connected": {
      "name": "",
      "number": ""
    },
    "accountcode": "",
    "dialplan": {
      "context": "sets",
      "exten": "242",
      "priority": 2
    },
    "creationtime": "2019-01-27T21:43:43.709-0500",
    "language": "en"
    },
  "asterisk_id": "08:00:27:27:bf:0e",
  "application": "zarniwoop"
}
>
```

Хорошо, теперь мы откроем еще один сеанс оболочки[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html#idm46178396682696), чтобы взаимодействовать с этим соединением, которое мы создали. Из этой новой оболочки выполните следующую команду:

```text
$ curl -v -u asterisk:чтобывыниделалинеиспользуйтеэтотпароль -X POST \
 "http://localhost:8088/ari/channels/1548643423.2/play?media=sound:believe-its-free" sd
```

Обратите внимание, что "`id` "из JSON, возвращаемый в сеансе `wscat`, должен использоваться после части `'channels/'` команды `curl`. Другими словами, вы должны сопоставить идентификатор канала в вашей команде с идентификатором канала, связанным с вашим вызовом. Таким образом, вы можете одновременно обрабатывать множество звонков.

### Работа с вашей средой ARI с использованием Swagger

Asterisk's ARI был разработан, чтобы быть совместимым со спецификацией Open API \(aka Swagger\) и это означает, что многие инструменты, совместимые с этой спецификацией, будут работать с ARI. Например, вы можете взаимодействовать с установкой AIR с помощью Swagger-UI, который будет полезен как для отладки, так и в качестве источника документации.

Во-первых, нам нужно будет открыть наш HTTP-сервер Asterisk в локальной сети \(в настоящее время он разрешает только соединения с 127.0.0.1\). В вашем файле _/etc/asterisk/http.conf_ мы привяжем HTTP-сервер к локальному IP-адресу машины "Asterisk":

```text
$ sudo vim /etc/asterisk/http.conf
; Включите встроенный HTTP-сервер и прослушивайте только соединения на локальном хосте.
[general]
enabled = yes
;bindaddr = 127.0.0.1 ; закомментируйте это
bindaddr = 172.29.1.57 ; LAN IP ВАШЕГО СЕРВЕРА ASTERISK
```

Далее нам нужно добавить строку в ваш файл _/etc/asterisk/ari.conf_:

```text
$ sudo vim /etc/asterisk/ari.conf
[general]
enabled = yes
pretty = yes
allowed_origins=http://ari.asterisk.org
...
```

Сохраните и перезагрузите модули `http` and `ari` в Asterisk:

```text
$ sudo asterisk -rx 'module reload http' ; sudo asterisk -rx 'module reload ari'
```

Теперь на рабочем столе разработчика откройте браузер и перейдите в раздел [http://ari.asterisk.org](http://ari.asterisk.org/).

Вы увидите страницу, похожую на Рисунок 19-1.

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 19-1. Swagger UI &#x434;&#x43B;&#x44F; ARI](.gitbook/assets/0%20%285%29.png)

Замените `localhost` на LAN IP-адрес вашего сервера Asterisk, а в поле api\_key введите Ваш ARI _`user:password`_ из _/etc/asterisk/ari.conf_ \(например, `asterisk:чтобывыниделалинеиспользуйтеэтотпароль`\). Если у вас есть все настройки верны, то вы будете вознаграждены с результатами как на Рисунке 19-2.

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 19-2. ARI Swagger](.gitbook/assets/1%20%285%29.png)

Вы видите полную документацию для вашего модуля ARI, и можете на самом деле так же передавать к нему запросы. Это очень полезно при отладке, и хвала людям Digium за это.

В качестве примера того, для чего это нужно, выберите пункт `endpoints:Endpoint resources`, нажмите кнопку `GET` рядом с `/endpoints`, и вы увидите экран, показанный на Рисунке 19-3.

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 19-3. Get endpoints](.gitbook/assets/2%20%284%29.png)

Ну, давай - жми на кнопку "Try it out!".

Обратите внимание на "`id`" канала в сеансе `wscat`, который вы хотите скопировать для использования в Swagger UI \(вы увидите несколько строк вывода JSON, связанных с вызовом\).

Выполните следующие действия по каналу через интерфейс Swagger UI: `POST: Answer` \(ответ на канал\), `POST: hold` \(поставить вызов на удержание\), `DELETE: hold` \(принять вызов из режима ожидания\). Обратите внимание на то, что происходит с каналом в каждом случае.

Использование этого Swagger UI также документировано в [Asterisk wiki](https://wiki.asterisk.org/wiki/display/AST/Using+Swagger+to+Drive+ARI).

Это значительно упростит процесс разработки и тестирования.

Хорошо, это быстрый старт. Давайте нырнем поглубже в ARI.

## Строительные блоки ARI

Есть три компонента, которые работают вместе для обеспечения ARI:

* RESTful интерфейс, через который внешнее приложение взаимодействует с Asterisk.
* WebSocket, который передает информацию обратно во внешнее приложение из Asterisk \(в формате JSON\).
* Приложение диалплана `Stasis()`, которое соединяет управление каналом с внешним приложением.

### REST

Термин _RESTful_ происходит от Representational State Transfer \(REST\), который является архитектурной моделью для веб-служб \(в отличие, скажем, от протокола\). Термин RESTful обычно относится к любому API, который обеспечивает взаимодействие через URL-адреса с данными, представленными в формате JSON.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html#idm46178396640584) Итак, все, что является "RESTfull", должно соответствовать ограничениям REST, но на практике может быть реализовано как более свободная интерпретация \(которая, если выполнит свою работу, действительно может быть достаточно хорошей\).

### WebSocket

Соединение WebSocket - это механизм, который осуществляет связь между внутренними компонентами Asterisk и интерфейсом RESTful. В Asterisk могут происходить события, которые клиент не инициировал, и WebSocket позволяет Asterisk сигнализировать об этих изменениях клиенту.

Встроенный HTTP-сервер Asterisk потенциально предоставляет другие сервисы через веб-интерфейс. Например, WebRTC также подключается через веб-сервер. Если вы вносите изменения или добавляете новые службы, убедитесь, что проверили не только элемент, над которым работаете, но и другие службы, работающие через тот же сервер, чтобы убедиться, что случайно не расстроили что-то другое.

### Stasis

The Stasis Message Bus allows the core of Asterisk to communicate events with other modules and components. It is mostly internal to Asterisk; however, in the case of ARI, a dialplan application named Stasis\(\) allows the dialplan to pass call control to your external ARI application.

The Stasis\(\) application itself is required in order to signal to the dialplan that call control is to be passed to the external program via ARI.

As of Asterisk 16, it is no longer necessary to write dialplan code to define a connection from an incoming channel to your ARI client application. Many developers in the Asterisk community write all their call control logic in external applications, and having to code up a few lines of dialplan just to pass channels to their app was seen as kludgy and confusing. They requested \(and developed\) a mechanism whereby Asterisk will create automatic dialplan to handle this function.

When the API is instantiated, the application reference in the URL—for example, our app zarniwoop—will trigger the automatic creation of a dialplan context named according to the app name \(in this case, \[stasis-zarniwoop\]\), including an extension that pattern matches everything. This extension will then pass all calls arriving in that context to Stasis\(zarniwoop\). You will need to associate your channels with the correct context \(context=stasis-zarniwoop\) in your PJSIP \(or other channel\) configuration tables, at which point calls to those channels will automatically be connected through Stasis\(\) to the client application.

If all this seems confusing, there’s no reason you need to stop using actual dialplan to handle this, as we did earlier in our quick-start example.

Understanding the workings of Stasis\(\) is generally not necessary unless you are going to be developing the Asterisk product itself \(i.e., joining the Asterisk development team and coding new capabilities into Asterisk\).

Typically, after your initial experimentation with ARI, you will want to implement a framework to help ease the work of developing your external application.

## Фреймворки

A production-grade application using ARI will benefit from the implementation of a framework to simplify development effort, add a layer of security, and provide a control environment.

There are several such libraries available. Which one you choose will in part be dictated by which language you prefer to use, and should also take into account whether the framework you’re interested in has an active community and is still being actively maintained.

The ones described next are listed in the Asterisk wiki. We examined the code repository for each, and while some projects are still actively maintained, others have not been updated in quite some time. If you are planning to implement one of these frameworks, you will need to do your own due diligence to ensure you can get support for it. In many cases, it may be worthwhile to reach out to the developers, and determine their consulting rates so you can ensure priority access to their time should you need it.

### ari-py \(and aioari\) for Python

The [ari-py framework](https://github.com/asterisk/ari-py) was written by Digium in 2013–2014, and as of this writing had not been updated since then. This framework builds on Asterisk’s Swagger.py client.

Shortly after the relase of ari-py, it was forked into the [aioari project](https://pypi.org/project/aioari/), which delivers an asynchronous version of ari-py. This code has been more steadily updated since then \(although as of this writing had not been updated since early 2018\). This framework should be included in your evaluation of a Python framework for ARI.

If you are looking to develop ARI applications in Python, one of these two frameworks may be what you are looking for. If you are looking to build a large ARI application, you will need to ensure that you have carefully tested the performance implications of using Python for what you are doing.

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### node-ari-client

For the JavaScript folks, there is a Node.js-based ARI framework that was first released in early 2014, and as of this writing is still being updated. It is based on the automatically generated API that comes from swagger-js.

For JavaScript/Node developers, this is where you’ll want to start: [https://github.com/asterisk/node-ari-client](https://github.com/asterisk/node-ari-client).

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### AsterNET.ARI

The Windows folks are not left out. The AsterNET.ARI project delivers a framework for .NET that augments the AsterNET project \(which also includes integration with Asterisk’s FastAGI and AMI interfaces\).

You can find the repository for AsterNET.ARI here: [https://github.com/skrusty/AsterNET.ARI](https://github.com/skrusty/AsterNET.ARI).

Digium has provided samples for this framework \(and others\) at [https://github.com/asterisk/ari-examples](https://github.com/asterisk/ari-examples).

### ari4java

The ari4java project is one of the most actively developed ARI frameworks we have found. It has been developed since 2013, and the repository was receiving commits at the same time as this writing.

If Java is your language, you will want to check out the ari4java repository at [https://github.com/l3nz/ari4java](https://github.com/l3nz/ari4java).

### phpari

The phpari project delivers an ARI framework for the PHP community. It has been developed since 2014, and the repository was still being updated as of this writing.

For the PHP fans, you’ll find the repository at [https://github.com/greenfieldtech-nirs/phpari](https://github.com/greenfieldtech-nirs/phpari).

### aricpp

If you’re used to writing in C++, there’s even an ARI project for you. The aricpp framework consists of header files only, so you can build its functions right into whatever you’re developing. This library has also been performance tested with SIPp, and while we don’t have any numbers on that, it seems to us that a compiled framework that has been performance tested is very much worth taking for a spin if you have the right skills.

One of the newer of the ARI frameworks, this project benefits from regular updates. Check it out at [https://github.com/daniele77/aricpp](https://github.com/daniele77/aricpp).

### asterisk-ari-client

Yes, Ruby also has an ARI framework.

You can find it at [https://github.com/svoboda-jan/asterisk-ari](https://github.com/svoboda-jan/asterisk-ari).

## Вывод

ARI предоставляет RESTful API текущего поколения, который может использоваться для разработки коммуникационных приложений с использованием популярных языков разработки. С его помощью опытный разработчик может использовать мощь самой успешной платформы АТС в истории. Это позволяет коммуникационным приложениям следующего поколения взаимодействовать с устаревшими телекоммуникационными протоколами и приложениями, что может оказаться очень полезным, поскольку мы все чаще призваны преодолевать разрыв между прошлым, настоящим и будущим коммуникационных технологий.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178403407976-marker) Хотя, честно говоря, в конфигурации нет особой сложности, если вы решите внедрить одну из платформ, что настоятельно рекомендуется для продакшена, и которую мы рассмотрим позже.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396697256-marker) Мы назвали приложение "zarniwoop", потому что “hello-world” использовалось в Digium wiki для ARI, и нам показалось, что лучше избегать перекрытия. Вы, конечно, можете назвать его как угодно.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396682696-marker) Если ваш компьютер имеет только один экран, то, вероятно, это то место, где вы задумаетесь, что неплохо было бы иметь их больше.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22idm46178396640584-marker) Строго говоря, REST - это гораздо больше, но на практике в наши дни не редкость предположить, что REST API будет основан на URL и JSON просто потому, что много сервисов представлены именно в этих форматах.

