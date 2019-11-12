# Глава 9. Интернационализация

> **Дэвид Дюффетт**
>
> _Английский? Кто должен тратить время на его изучение? Я никогда не поеду в Англию!_
>
> -- Дэн Касталланета

Телефония - одна из тех сфер жизни, где люди не любят сюрпризов, будь они дома или на работе. Когда люди пользуются телефонами, все, что выходит за рамки нормы, не оправдывается, и, как человек, который, вероятно занимается поставками телефонных систем, вы будете знать, что неудовлетворенные ожидания могут привести к неописуемым страданиям с точки зрения дополнительной работы, потерянных денег и других проблем, связанных с недовольством клиентов.

Помимо того, что пользовательский интерфейс соответствует тому, что ожидают пользователи, также необходимо, чтобы ваш Asterisk чувствовал себя «как дома». Например, если исходящий вызов сделан по аналоговой линии \(FXO\), Asterisk будет нужно интерпретировать тоны, которые он «слышит» на линии \(занято, вызов и т.д.\).

По умолчанию \(а возможно, как и следовало бы ожидать, поскольку он был “рожден в США”\), Asterisk настроен на работу в Северной Америке. Однако, поскольку Asterisk развертывается во многих местах и \(к счастью\) люди со всего мира вносят свой вклад в него, вполне возможно настроить Asterisk для правильной работы практически в любом месте, где вы решите его развернуть.

Если вы читали эту книгу с самого начала - глава за главой, вы уже сделали некоторый выбор во время установки и начальной конфигурации, которая настроила ваш Asterisk для работы в вашем регионе \(и оправдает ожидания ваших клиентов\).

Довольно много глав в этой книге содержат информацию, которая поможет вам интернационализировать[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html#idm46178407219928) или \(возможно, более правильно\) локализовать вашу реализацию Asterisk. Цель этой главы - обеспечить единое место, куда можно будет ссылаться, обсуждать и объяснять все аспекты изменений, которые необходимо внести в телефонную систему на базе Asterisk в этом контексте. Причина использования фразы "телефонная система на основе Asterisk", а не просто” Asterisk", заключается в том, что некоторые изменения необходимо будет внести в другие части системы \(IP-телефоны, ATAs и т. д.\), в то время как другие изменения будут реализованы в конфигурационных файлах Asterisk и DAHDI.

Давайте начнем с составления списка \(не в определенном порядке\) вещей, которые возможно потребуется изменить для оптимизации вашей телефонной системы на основе Asterisk для данного местоположения за пределами Северной Америки. Вы можете выкрикнуть что-нибудь ещё, если хотите.…

* Язык/акцент подсказок
* Физическое подключение для интерфейсов ТфОП \(FXO, BRI, PRI\)
* Тоны, слышимые пользователями IP-телефонов и/или ATA
* Формат идентификатора вызывающего абонента \(CallerID\), передаваемый и/или принимаемый аналоговыми интерфейсами
* Сигналы для аналоговых интерфейсов, подаваемые или определяемые Asterisk
* Формат меток времени/даты для голосовой почты
* Способ, которым вышеуказанные метки времени/даты объявляются системой Asterisk
* Шаблоны в диалплане \(IP-телефонов, ATA и самой Asterisk, если вы используете образец диалплана\) 
* Способ указания аналоговому устройству об ожидании голосовой почты \(MWI\)
* Тоны, подаваемые абонентам Asterisk \(они вступают в игру, когда пользователь находится “внутри " системы; например, тоны, услышанные во время трансфера вызова\)

Мы рассмотрим все в этом списке, приняв стратегию работы от внешнего края системы к самому ядру \(самой Asterisk\). Мы закончим с удобным контрольным списком того, что вам может понадобится изменить и где это сделать.

Хотя принципы, описанные в этой главе, позволят вам адаптировать установку Asterisk специально для вашего региона \(или для вашего клиента\), для обеспечения согласованности все наши примеры будут сосредоточены на том, как адаптировать Asterisk для одного региона: Соединенного Королевства.

## Внешние устройства по отношению к серверу Asterisk

Существуют огромные различия между хорошим старомодным аналоговым телефоном и любым из большого количества IP-телефонов, и нам нужно подобрать одно из действительно фундаментальных различий, чтобы пролить свет на следующее объяснение, которое охватывает настройки, которые нам, возможно, придется изменить на устройствах, внешних по отношению к Asterisk, таких как IP-телефоны.

Вы когда-нибудь задумывались о том, что аналоговый телефон - это совершенно немое устройство \(мы знаем, что базовая модель очень, очень дешевая\), которое должно подключаться к интеллектуальной сети \(ТфОП\), тогда как IP-телефон \(например, SIP или IAX2\) - это очень умное устройство, которое подключается к немой сети \(Интернет или любая обычная IP-сеть\)? Рисунки 9-1 и 9-2 проиллюстрируют разницу.

![](.gitbook/assets/0%20%289%29.png)

**Рисунок 9-1. Старые времена: немые устройства подключаются к умной сети**

![](.gitbook/assets/1%20%286%29.png)

**Рисунок 9-2. Ситуация на сегодняшний день: умные устройства подключаются через немую сеть**

Можем ли мы взять два аналоговых телефона, подключить их непосредственно друг к другу и иметь функциональность, которую мы обычно связываем с обычным телефоном? Нет, конечно нет, потому что сеть предоставляет все: фактическое питание телефона, сигнал вызова \(от местной станции или центрального офиса\), информацию об идентификаторе вызывающего абонента \(CallerID\), сигнал вызова \(от удаленной \[ближайшего к телефону назначения\] станции или ЦО\), всю необходимую сигнализацию и так далее.

И наоборот, можем ли мы взять два IP-телефона, подключить их непосредственно друг к другу и получить некоторую разумную функциональность? Конечно можем, потому что весь интеллект находится внутри самих IP—телефонов - они обеспечивают тоны, которые мы слышим \(сигнал вызова, звонок, занято\) и запускают протокол, который выполняет всю необходимую сигнализацию \(обычно SIP\). Фактически, вы можете попробовать это для себя - большинство средних IP-телефонов имеют встроенный коммутатор Ethernet, поэтому вы можете подключить два IP-телефона непосредственно друг к другу с помощью обычного \(прямого\) кабеля Ethernet или просто подключить их через обычный коммутатор. Они должны иметь фиксированные IP-адреса в отсутствие DHCP-сервера, и вы сможете набрать IP-адрес другого телефона, просто используя клавишу \* для точек в адресе.

Рисунок 9-2 указывает на тот факт, что на IP-телефоне мы несем ответственность за настройку всех тонов, которые предоставила бы сеть в былые времена. Это можно сделать одним \(по крайней мере\) из двух способов. Первый заключается в настройке тонов, предоставляемых IP-телефоном на собственном веб-интерфейсе устройства. Вы делаете это, просматривая IP-адрес телефона \(IP-адрес обычно можно получить с помощью опции меню на телефоне\), а затем выбрав соответствующие параметры. Например, на IP-телефоне Yealink тоны устанавливаются на странице веб-графического интерфейса _Телефон_ под вкладкой _Тоны_ \(где вы найдете список различных типов тонов, которые можно изменить — в случае Yealink это набор, КПВ, занято, перегрузка, ожидание вызова, повторный вызов, запись, информация, заикание, сообщение и автоответ\).

Другой способ, которым эта конфигурация может быть применена - это автоматическое предоставление телефону этих настроек. Полное объяснение механизма автопровижинга выходит за рамки этой книги, но как правило вы можете настроить тоны в соответствующих атрибутах необходимых элементов в XML-файле.

В то время как мы меняем настройки на IP-телефонах, есть еще две вещи, которые может потребоваться изменить, чтобы телефоны выглядели правильно и функционировали как часть системы.

Большинство телефонов отображают время в режиме ожидания, и, поскольку многие люди находят это особенно раздражающим, когда их телефоны показывают неправильное время, мы должны убедиться, что отображается правильное местное время. Должно быть довольно легко найти соответствующую страницу веб-интерфейса \(или атрибутов XML\) для указания сервера сингхронизации времени. Вы также обнаружите что есть настройки для перехода на летнее время и другие важные вещи.

Последнее, что нужно изменить - это потенциальный showstopper, когда речь идет о телефонном звонке - диалплан. Мы говорим не о диалплане, который находится в _/etc/asterisk/extensions.conf_, а о диалплане телефона. Не все понимают что IP-телефоны также имеют схемы набора номеров, хотя эти диалпланы больше связаны с тем, какие строки набора разрешены, чем с тем, что делать с данным набором.

Общее правило, по-видимому, заключается в том, что если вы набираете при положенной трубке - встроенная схема набора номера проигнорируется, но если вы поднимаете трубку - в игру вступает диалплан телефона и может случиться так, что диалплан не позволит набрать необходимую строку. Хотя эта проблема может проявляться в отказе телефона передавать определенные типы номеров в Asterisk, она также может повлиять на любые коды функций, которые вы планируете использовать. Это может быть легко исправлено путем поиска номера модели телефона вместе с «UK dialplan» \(или конкретным нужным вам регионом\) или вы можете перейти на соответствующую страницу в веб-интерфейсе пользователя и там либо вручную настроить диалплан, либо выбрать страну из выпадающего списка \(в зависимости от типа телефона, с которым вы работаете\).

Предварительное обсуждение конфигурации IP-телефона также относится к любым аналоговым телефонным адаптерам \(ATA\), которые вы планируете использовать, в частности к тем, которые поддерживают интерфейс FXS. Кроме того, может потребоваться указать некоторые электрические характеристики телефонного интерфейса, такие как линейное напряжение и импеданс, а также формат идентификатора вызывающего абонента, который будет работать с локальными телефонами. Все, что отличается - это способ получения IP-адреса для веб-интерфейса - обычно это делается набором определенного кода на подключенном аналоговом телефоне, что приводит к тому, что IP-адрес произносится вызывающему абоненту. 

Конечно, ATA также может иметь интерфейс FXO, который также должен быть настроен для правильного взаимодействия с аналоговой линией, предоставляемой в вашем регионе. Типы настроек, которые необходимо изменить, аналогичны интерфейсу FXS. 

Что делать, если вы подключаете аналоговый телефон или линию к карте Digium? Мы рассмотрим это в следующий раз.

## Подключение ТфОП, DAHDI, карт Digium и аналоговых телефонов

Прежде чем мы перейдем к конфигурации DAHDI и Asterisk, нам нужно физически подключиться к ТфОП. К сожалению, общемировых стандартов для пордобных соединений не существует; на самом деле, даже в разных частях одной страны часто существуют различия. 

Primary Rate Interfaces \(PRI\) обычно оканчиваются соединением RJ45 в настоящее время, хотя импеданс соединений может варьироваться. В некоторых странах \(в частности, в Южной Америке\) все еще можно найти PRI, обжатый двумя разъемами BNC: один для передачи и один для приема. 

Проще говоря PRI, оконеченный RJ45, будет соединением ISDN, а если вы обнаружите, что соединение выполнено парой разъемов BNC \(push-and-twist coaxial connectors\), велика вероятность что вы имеете дело с более старым протоколом на основе CAS \(например, MFCR2\). 

На Рисунке 9-3 показан адаптер, необходимый в том случае, если ваша телефонная компания поставила разъемы BNC \(карты Sangoma/Digium требуют подключения RJ45\). Он называется _balun_, поскольку преобразует из сбалансированного соединения \(RJ45\) в несбалансированное соединение \(BNCs\) в дополнение к изменению импеданса соединения.

{% hint style="info" %}
#### Примечание

Basic Rate Interfaces \(BRIs\) распроастранены в континентальной Европе и почти всегда поставляются через соединение RJ45.
{% endhint %}

#### 

![](.gitbook/assets/2%20%285%29.png)

#### Рисунок 9-3. Balun

Аналоговые соединения сильно различаются в зависимости от местоположения - вы должны знать, какой тип разъема используется в вашей местности. Важно помнить, что аналоговая линия - это только два провода, и они должны подключаться к двум средним контактам разъема RJ11, который входит в плату Digium, другой конец является локальным. На Рисунке 9-4 показан коннектор, используемый в Великобритании, где два провода подключены к контактам 2 и 5.

![](.gitbook/assets/3%20%281%29.png)

#### Рисунок 9-4. Штекер BT, используемый для аналоговых соединений ТфОП в Великобритании \(обратите внимание, присутствуют только контакты 2–5\)

Интерфейс аппаратного устройства Digium Asterisk \(Digium Asterisk Hardware Device Interface\) или DAHDI на самом деле охватывает несколько вещей. Он содержит драйверы ядра для плат адаптеров телефонии, которые работают в рамках DAHDI, а также утилиты автоматической настройки и инструменты тестирования. Эти части содержатся в двух отдельных пакетах \(_dahdi-linux_ и _dahdi-tools_\), но мы также можем использовать один полный пакет, который называется _dahdi-linux-complete_. Все три пакета доступны [на сайте Digium](http://downloads.digium.com/pub/telephony/). 

После того, как вы установили тип соединения PRI, предоставленного вам телекоммуникационным оператором, вам понадобятся некоторые дополнительные сведения для правильной настройки DAHDI и Asterisk \(например, является ли соединение ISDN или протоколом на основе CAS\). Опять же, вы найдете их в [Главе 7](glava-07.md).

### DAHDI Drivers

The connections where some real localization will need to take place are those of analog interfaces. For the purposes of configuring your Asterisk-based telephone system to work best in a given locality, you will first need to specifically configure some low-level aspects of the way the Digium card interacts with the connected device or line. This is done through the DAHDI kernel driver\(s\), in a file called /etc/dahdi/system.conf.

In the following lines \(taken from the sample configuration that you get with a fresh install of DAHDI\), you will find both the loadzone and defaultzone settings. The loadzone setting allows you to choose which tone set\(s\) the card will both generate \(to feed to analog telephones\) and recognize \(on the connected analog telephone lines\):

```text
# Tone Zone Data
# ^^^^^^^^^^^^^^
# Finally, you can preload some tone zones, to prevent them from getting
# overwritten by other users (if you allow non-root users to open /dev/dahdi/*
# interfaces anyway). Also this means they won't have to be loaded at runtime.
# The format is "loadzone=<zone>" where the zone is a two-letter country code.
#
# You may also specify a default zone with "defaultzone=<zone>" where zone
# is a two-letter country code.
#
# An up-to-date list of the zones can be found in the file zonedata.c
#
loadzone = us
#loadzone = us-old
#loadzone=gr
#loadzone=it
#loadzone=fr
#loadzone=de
#loadzone=uk
#loadzone=fi
#loadzone=jp
#loadzone=sp
#loadzone=no
#loadzone=hu
#loadzone=lt
#loadzone=pl
defaultzone=us
#
```

{% hint style="info" %}
#### Tip

The /etc/dahdi/system.conf file uses the hash symbol \(\#\) to indicate a comment instead of a semicolon \(;\) like the files in /etc/asterisk.
{% endhint %}

Although it is possible to load a number of different tone sets \(you can see all the sets of tones in detail in zonedata.c\) and to switch between them, in most practical situations you will only need:

```text
loadzone=uk # to load the tone set
defaultzone=uk # to default DAHDI to using that set
```

…or whichever tones you need for your region.

If you perform a dahdi\_genconf to automatically \(or should that be auto-magically?\) configure your DAHDI adapters, you will notice that the newly generated /etc/dahdi/system.conf will have defaulted both loadzone and defaultzone to being us. Despite the warnings not to hand-edit the file, it is fine to change these settings to what you need.

In case you were wondering how we tell whether there are any voicemails in the mailbox associated with the channel an analog phone is plugged into, it is done with a stuttered dialtone. The format of this stuttered dialtone is decided by the loadzone/defaultzone combination you have used.

As a quick aside, analog phones that have a message-waiting indicator \(e.g., an LED or lamp that flashes to indicate new voicemail\) achieve this by automatically going off-hook periodically and listening for the stuttered dialtone. You can witness this by watching the Asterisk command line to see the DAHDI channel go active \(if you have nothing better to do!\).

That’s it at the DAHDI level. We chose the protocol\(s\) for PRI or BRI connections, the type of signaling for the analog channels \(all covered in [Chapter 7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22asterisk-OutsideConn)\), and the tones for the analog connections that have just been discussed.

The relationship between Linux, DAHDI, and Asterisk \(and therefore /etc/dahdi/system.conf and /etc/asterisk/chan\_dahdi.conf\) is shown in [Figure 9-5](https://github.com/Krotesk1/definitive-guide-5th/tree/25dd8a8bd31ab9128242e9ca42e5fc842f433f2f/9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-dahdi-relationship/README.md).

{% hint style="info" %}
#### Tip

Once you have completed your configuration at the DAHDI level \(in /etc/dahdi/system.conf\), you need to perform a dahdi\_cfg -vvv to have DAHDI reread the configuration. This is also a good time to use dahdi\_tool to check that everything appears to be in order at the Linux level.

This way, if things do not work properly after you have configured Asterisk to work with the DAHDI adapters, you can be sure that the problem is confined to chan\_dahdi.conf \(or an \#included dahdi-channels.conf if you are using this part of the dahdi\_genconf output\).
{% endhint %}

![](.gitbook/assets/4%20%281%29.png)

#### Figure 9-5. The relationship between Linux, DAHDI, and Asterisk

## Internationalization Within Asterisk

With everything set at the Linux level, we now only need to configure Asterisk to make use of the channels we just enabled at the Linux level and to customize the way that Asterisk interprets and generates information that comes in from, or goes out over, these channels. This work is done in /etc/asterisk/chan\_dahdi.conf.

In this file we will not only tell Asterisk what sort of channels we have \(these settings will fit with what we already did in DAHDI\), but also configure a number of things that will ensure Asterisk is well suited to its new home.

### Caller ID

A key component of this change is caller ID. While caller ID delivery methods are pretty much standard within the BRI and PRI world, they vary widely in the analog world; thus, if you plugged an American analog phone into the UK telephone network, it would actually work as a phone, but caller ID information would not be displayed. This is because that information is transmitted in different ways in different places around the world, and an American phone would be looking for caller ID signaling in the US format, while the UK telephone network would be supplying it in the UK format \(if it is enabled—caller ID is not standard in the UK; you have to ask for and sometimes even pay for, it!\).

Not only is the format different, but the method of telling a telephone \(or Asterisk\) to look out for the caller ID may vary from place to place, too. This is important, as we do not want Asterisk to waste time looking for caller ID information if it is not being presented on the line.

Again, Asterisk defaults to the North American caller ID format \(no entries in /etc/asterisk/chan\_dahdi.conf describe this, it’s just the default\), and in order to change it we will need to make some entries that describe the technical details of the caller ID system. In the case of the UK, the delivery of caller ID information is signaled by a polarity reversal on the telephone line \(in other words, the A and B legs of the pair of telephone wires are temporarily switched over\), and the actual caller ID information is delivered in a format known as V.23 \(frequency shift keying, or FSK\). So the entries in chan\_dahdi.conf to receive UK-style caller ID on any FXO interfaces will look like this:

```text
cidstart=polarity ; the delivery of caller ID will be
                  ; signaled by a polarity reversal
cidsignalling=v23 ; the delivery of the called ID information
                  ; will be in V23 format
```

Of course, you may also need to send caller ID using the same local signaling information to any analog phones that are connected to FXS interfaces, and one more entry may be needed, as in some locations the caller ID information is sent after a specified number of rings. If this is the case, you can use this entry:

```text
sendcalleridafter=2
```

Before you can make these entries, you will need to establish the details of your local caller ID system \(someone from your local telco or Google could be your friend here, but there is also some good information in the sample /etc/asterisk/chan\_dahdi.conf file\).

### Language and/or Accent of Prompts

As you may know, the prompts \(or recordings\) that Asterisk will use are stored in /var/lib/asterisk/sounds. In older versions of Asterisk all the sounds were in this actual directory, but these days you will find a number of subdirectories that allow the use of different languages or accents. The names of these subdirectories are arbitrary; you can call them whatever you want.

Note that the filenames in these directories must be what Asterisk is expecting—for example, in /var/lib/asterisk/sound/en, the file hello.gsm would contain the word “Hello” \(spoken by the lovely Allison\), whereas hello.gsm in /var/lib/asterisk/sounds/es \(for Spanish in this case\) would contain the word “Hola” \(spoken by the Spanish equivalent of the lovely Allison[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407073864)\).

The default directory used is /var/lib/asterisk/sounds/en, so how do you change that?

There are two ways. One is to set the language in the channel configuration file that calls are arriving through using the language directive. For example, the line:

```text
language=en_UK
```

placed in chan\_dahdi.conf, sip.conf, and so on \(to apply generally, or for just a given channel or profile\) will tell Asterisk to use sound files found in /var/lib/asterisk/sounds/en\_UK \(which could contain British-accented prompts\) for all calls that come in through those channels.

The other way is to change the language during a phone call through the dialplan. This \(along with many attributes of an individual call\) can be set using the CHANNEL\(\) dialplan function. See [Chapter 10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-DP-Deeper) for a full treatment of dialplan functions.

The following example would allow the caller to choose one of three languages in which to continue the call:

```text
; gives the choice of (1) French, (2) Spanish, or (3) German
exten => s,1,Background(choose-language)
same => n,WaitExten(5)
exten => 1,1,Set(CHANNEL(language)=fr)
exten => 2,1,Set(CHANNEL(language)=es)
exten => 3,1,Set(CHANNEL(language)=de)
; the next priority for extensions 1, 2, or 3 would be handled here
exten => _[123],n,Goto(menu,s,1)
```

If the caller pressed 1, sounds would be played from /var/lib/asterisk/sounds/fr; if he pressed 2, the sounds would come from /var/lib/asterisk/sounds/es; and so on.

As already mentioned, the names of these directories are arbitrary and do not need to be only two characters long—the main thing is that you match the name of the subdirectory you have created in the language directive in the channel configuration, or when you set the CHANNEL\(language\) argument in the dialplan.

### Time/Date Stamps and Pronunciation

Asterisk uses the Linux system time from the host server, as you would expect, but we may have users of the system who are in different time zones, or even in different countries. Voicemail is where the rubber hits the road, as this is where users come into contact with time/date stamp information.

Consider a scenario where some users of the system are based in the US, while others are in the UK.

As well as the time difference, another thing to consider is the way people in the two locations are used to hearing date and time information—in the US, dates are usually ordered month, day, year, and times are specified in 12-hour clock format \(e.g., 2:54 P.M.\).

In contrast, in the UK dates are ordered day, month, year, and times are often specified in 24-hour clock format \(14:54 hrs\)—although some people in the UK prefer 12-hour clock format, so we will cover that, too.

Since all these things are connected to voicemail, you would be right to guess that we configure it in /etc/asterisk/voicemail.conf—specifically, in the \[zonemessages\] section of the file.

Here is the \[zonemessages\] part of the sample voicemail.conf file that comes with Asterisk, with UK24 \(for UK people that like 24-hour clock format times\) and UK12 \(for UK people that prefer 12-hour clock format\) zones added:

```text
[zonemessages]
; Users may be located in different time zones, or may have different
; message announcements for their introductory message when they enter
; the voicemail system. Set the message and the time zone each user
; hears here. Set the user into one of these zones with the tz=attribute
; in the options field of the mailbox. Of course, language substitution
; still applies here so you may have several directory trees that have
; alternate language choices.
;
; Look in /usr/share/zoneinfo/ for names of timezones.
; Look at the manual page for strftime for a quick tutorial on how the
; variable substitution is done on the values below.
;
; Supported values:
; 'filename' filename of a soundfile (single ticks around the filename
; required)
; ${VAR} variable substitution
; A or a Day of week (Saturday, Sunday, ...)
; B or b or h Month name (January, February, ...)
; d or e numeric day of month (first, second, ... thirty-first)
; Y Year
; I or l Hour, 12 hour clock
; H Hour, 24 hour clock (single digit hours preceded by "oh")
; k Hour, 24 hour clock (single digit hours NOT preceded by "oh")
; M Minute, with 00 pronounced as "o'clock"
; N Minute, with 00 pronounced as "hundred" (US military time)
; P or p AM or PM
; Q "today", "yesterday" or ABdY
; (*note: not standard strftime value)
; q " (for today), "yesterday", weekday, or ABdY
; (*note: not standard strftime value)
; R 24 hour time, including minute
;
eastern=America/New_York|'vm-received' Q 'digits/at' IMp
central=America/Chicago|'vm-received' Q 'digits/at' IMp
central24=America/Chicago|'vm-received' q 'digits/at' H N 'hours'
military=Zulu|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z_p'
european=Europe/Copenhagen|'vm-received' a d b 'digits/at' HM
UK24=Europe/London|'vm-received' q 'digits/at' H N 'hours'
UK12=Europe/London|'vm-received' Q 'digits/at' IMp
```

These zones not only specify a time, but also dictate the way times and dates are ordered and read out.

Having created these zones, we can go to the voicemail context part of voicemail.conf to associate the appropriate mailboxes with the correct zones:

```text
[default]
4001 => 1234,Russell Bryant,rb@shifteight.org,,|tz=central
4002 => 4444,David Duffett,dd@shifteight.org,,|tz=UK24
4003 => 4450,Mary Poppins,mp@shifteight.org,,|tz=UK12|attach=yes
```

As you can see, when we declare a mailbox, we also \(optionally\) associate it with a particular zone. Full details on voicemail can be found in [Chapter 8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22asterisk-Voicemail).

The last thing to localize in our Asterisk configuration is the tones played to callers by Asterisk once they are inside the system \(e.g., the tones a caller hears during a transfer\).

As identified earlier in this chapter, the initial tones that people hear when they are calling into the system will come from the IP phone, or from DAHDI for analog channels.

These tones are set in /etc/asterisk/indications.conf. Here is a part of the sample file, where you can see a given region specified by the country directive. We just need to change the country code as appropriate:

```text
;
; indications.conf
;
; Configuration file for location specific tone indications
;
; NOTE:
; When adding countries to this file, please keep them in alphabetical
; order according to the 2-character country codes!
;
; The [general] category is for certain global variables.
; All other categories are interpreted as location specific indications
;
[general]
country=uk ; default is US, so we have changed it to UK
```

Your dialplan will need to reflect the numbering scheme for your region. If you do not already know the scheme for your area, your local telecoms regulator will usually be able to supply details of the plan. Also, the example dialplan in /etc/asterisk/extensions.conf is, of course, packed with North American numbers and patterns.

## Conclusion—Easy Reference Cheat Sheet

As you can now see, there are quite a few things to change in order to fully localize your Asterisk-based telephone system, and not all of them are in the Asterisk, or even DAHDI, configuration—some things need to be changed on the connected IP phones or ATAs themselves.

Before we leave the chapter, have a look at [Table 9-1](https://github.com/Krotesk1/definitive-guide-5th/tree/25dd8a8bd31ab9128242e9ca42e5fc842f433f2f/9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22internationalization_cheat/README.md): a cheat sheet for what to change and where to change it, for your future reference.

Table 9-1. Internationalization cheat sheet

<table>
  <thead>
    <tr>
      <th style="text-align:left">What to change</th>
      <th style="text-align:left">Where to change it</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Call progress tones</td>
      <td style="text-align:left">
        <p></p>
        <ul>
          <li>IP phones&#x2014;on the phone itself</li>
          <li>ATAs&#x2014;on the ATA itself</li>
          <li>Analog phones&#x2014;DAHDI (<em>/etc/dahdi/system.conf</em>)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Type of PRI/BRI and protocol</td>
      <td style="text-align:left">DAHDI&#x2014;<em>/etc/dahdi/system.conf</em> and <em>/etc/asterisk/chan_dahdi.conf</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Physical PSTN connections</td>
      <td style="text-align:left">
        <p></p>
        <ul>
          <li>Balun if required for PRI</li>
          <li>Get the analog pair to middle two pins of the RJ11 connecting to the Digium
            card</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Caller ID on analog circuits</td>
      <td style="text-align:left">Asterisk&#x2014;<em>/etc/asterisk/chan_dahdi.conf</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Prompt language and/or accent</td>
      <td style="text-align:left">
        <p></p>
        <ul>
          <li>Channel&#x2014;<em>/etc/asterisk/sip.conf</em>, <em>/etc/asterisk/iax.conf</em>, <em>/etc/asterisk/chan_dahdi.conf</em>,
            etc.</li>
          <li>Dialplan&#x2014;<code>CHANNEL(language)</code> function</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Voicemail time/date stamps and pronunciation</td>
      <td style="text-align:left">Asterisk&#x2014;<em>/etc/asterisk/voicemail.conf</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Tones delivered by Asterisk</td>
      <td style="text-align:left">Asterisk&#x2014;<em>/etc/asterisk/indications.conf</em>
      </td>
    </tr>
  </tbody>
</table>May all your Asterisk deployments feel at home…

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407219928-marker) i18n is a term used to abbreviate the word internationalization, due to its length. The format is &lt;first\_letter&gt;&lt;number&gt;&lt;last\_letter&gt;, where &lt;number&gt; is the number of letters between the first and last letters. Other words, such as localization \(L10n\) and modularization \(m12n\), have also found a home with this scheme, which Leif finds a little bit ridiculous. More information can be found in the [W3C glossary online](http://www.w3.org/2001/12/Glossary%22%20/l%20%22I18N).

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407073864-marker) Who is, in fact, the same Allison who does the English prompts; June Wallack does the French prompts. The male Australian-accented prompts are done by Cameron Twomey. All voiceover talent are available to record additional prompts as well. See the [Digium IVR page](http://www.digium.com/en/products/ivr/) for more information.

