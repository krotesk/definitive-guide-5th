# Глава 13. Состояния устройств

> Среди беспорядка найдите простоту.
>
> --_Albert Einstein_

Часто бывает полезно иметь возможность определить состояние устройств, подключенных к телефонной системе. Например, оператору в приемной может потребоваться видеть статусы всех сотрудников офиса, чтобы определить, может ли кто-то принять телефонный звонок. Сам Asterisk нуждается в такой же информации. В качестве другого примера, если вы создали очередь вызовов, как описано в [Главе 12](glava-15.md), Asterisk должен знать, когда агент становится доступен, чтобы можно было отправить другой вызов. В этой главе рассматриваются понятия состояний устройства в Asterisk, а также способы использования и доступа устройств и приложений к этой информации.

## Состояния устройств

Существует две категории устройств, для которых Asterisk предоставляет информацию о состоянии: канальные устройства (такие как конечные точки PJSIP) и виртуальные (являющиеся встроенными службами, которые можно отслеживать, например конференц-залы).

Чтобы ссылаться на состояние _канала_, Вы делаете это точно так же, как с `Dial()`, например `DEVICE_STATE(PJSIP/000f300B0B02)`, тогда как для ссылки на состояние _виртуального устройства_ используется формат `тип виртуального устройства:идентификатор`, например `DEVICE_STATE(ConfBridge:1234)`.

Виртуальные устройства включают вещи, которые находятся внутри Asterisk, но предоставляют полезную информацию о состоянии (см. [Таблицу 13-1](13.%20Device%20States%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22table-virtualDevices)\).

Таблица 13-1. Устройства, для которых Asterisk может предоставлять информацию о состоянии

| Устройство | Описание |
| :--- | :--- |
| `PJSIP/channel name` | Многие каналы позволяют контролировать их состояние, но канал PJSIP предлагает на сегодняшний день наибольшее количество полезных данных; таким образом, мониторинг SIP-устройств является наиболее распространенным использованием `DEVICE_STATE`. |
| `ConfBridge:conference bridge` | Состояние конференц-моста MeetMe. Состояние будет отражать, имеются ли в настоящее время участники в конференц-зале. Более подробную информацию об использовании `MeetMe()` для конференц-связи можно найти в [Главе 11](glava-11.md). |
| `Custom:custom name` | Пользовательские состояния устройств. Эти состояния имеют пользовательские имена и изменяются с помощью функции `DEVICE_STATE()`. Пример использования можно найти в разделе ["Использование пользовательских состояний устройств"](glava-13.md#Использование-пользовательских-состояний-устройств). |
| `Park:exten@context` | Состояние слота парковки вызова. Информация о состоянии будет отражать, припаркован ли вызывающий абонент в данный момент на этом добавочном номере. Дополнительную информацию о парковке вызовов в Asterisk можно найти в ["Парковке вызовов"](glava-11.md#Парковка-вызовов). |
| `Calendar:calendar name` | Состояние календаря. Asterisk будет использовать содержимое названного календаря, чтобы установить состояние `available` или `busy`. |

### Проверка состояний устройств

Функция диалплана `DEVICE_STATE()` считывает текущее состояние устройства.
```
exten => 7012,1,Answer()
   same => n,Set(DeviceIdent=PJSIP/000f300B0B02)
   same => n,Verbose(3,${DeviceIdent} is ${DEVICE_STATE($DeviceIdent})})
   same => n,Hangup()
```
Если мы вызываем расширение 7012 с того же устройства, на котором проверяем состояние, в консоли Asterisk появляется следующее подробное сообщение:
```
 -- PJSIP/000f300B0B02 is INUSE
```
---
#### Примечание

В [Главе 17](glava-17.md) обсуждается Asterisk Manager Interface (AMI). Действие диспетчера `GetVar` можно использовать для получения значения состояния устройства из внешней программы. Вы можете использовать его для получения значения обычной переменной или для возврата значения из функции диалплана, например `DEVICE_STATE()`.

---

Вот список значений, которые вернет функция `DEVICE_STATE()` (в зависимости, конечно, от того, что было найдено):

* `UNKNOWN`
* `NOT_INUSE`
* `INUSE`
* `BUSY`
* `INVALID`
* `UNAVAILABLE`
* `RINGING`
* `RINGINUSE`
* `ONHOLD`

Эта информация может затем использоваться в диалплане для принятия решений о потоке вызовов (например, локальный канал вызова агента может использовать эту информацию для определения того, что телефон агента находится на вызове по другой линии, и таким образом отклонить вызов, чтобы тот вернулся в очередь).

## Состояния расширения используя директиву hint

Состояние расширения - это механизм диалплана, который Asterisk использует, чтобы разрешить SIP-устройствам подписываться на информацию о присутствии. Например, телефон в приемной может иметь модуль Busy Lamp Field (BLF), содержащий кнопки, которые будут использоваться для отображения состояний различных телефонов в офисе. Телефон с BLF будет отправлять запросы на подписку, чтобы сообщить Asterisk, с каких устройств он хочет получать информацию о присутствии. В диалплане мы используем директиву hint для определения сопоставления между расширением и одним или несколькими устройствами.

### Хинты

Чтобы определить хинт в диалплане, вместо приоритета используется ключевое слово `hint`.
```
[hints]
;exten = <extension>,hint,<device state id>[& <more dev state id],<presence state id>

exten => 100,hint,${UserA_DeskPhone}

exten => 221,hint,ConfBridge:221
```
Часто вы можете увидеть хинты, определенные в том же разделе диалплана, что и обычные расширения. Это может сделать диалплан немного визуально загроможденным, и это также предполагает, что хинт так или иначе связан с набираемым добавочным номером, что на самом деле не так.
```
[sets]

exten => 100,hint,${UserA_DeskPhone}
exten => 100,1,Gosub(subDialUser,${EXTEN},1(${UserA_DeskPhone},${EXTEN},default,22))
exten => 101,hint,${UserA_SoftPhone}
exten => 101,1,Gosub(subDialUser,${EXTEN},1(${UserA_SoftPhone},${EXTEN},default,23))
exten => 102,hint,${UserB_DeskPhone}
exten => 102,1,Gosub(subDialUser,${EXTEN},1(${UserB_DeskPhone},${EXTEN},default,26))
exten => 103,hint,${UserB_SoftPhone}
exten => 103,1,Gosub(subDialUser,${EXTEN},1(${UserB_SoftPhone},${EXTEN},default,24))

exten => 110,1,Dial(${UserA_DeskPhone}&${UserA_SoftPhone}&${UserB_SoftPhone})
```
В нашем примере мы сделали прямую взаимосвязь между добавочным номером подсказки и набираемым добавочным номером, хотя этого не требуется.

### Проверка состояний расширения

Самый простой способ проверить текущее состояние хинтов расширений - через CLI Asterisk. Команда `core show hints` отобразит все настроенные в данный момент хинты:
```
*CLI> core show hints
    -= Registered Asterisk Dial Plan Hints =-
100@hints : PJSIP/0000f30A0A01   State:Unavailable Presence:not_set Watchers 0
101@hints : PJSIP/SOFTPHONE_A    State:Unavailable Presence:not_set Watchers 0
102@hints : PJSIP/0000f30B0B02   State:Unavailable Presence:not_set Watchers 0
103@hints : PJSIP/SOFTPHONE_B    State:Unavailable Presence:not_set Watchers 0
110@hints : PJSIP/0000f30A0A01&P State:Unavailable Presence:not_set Watchers 0
221@hints : ConfBridge:221       State:Unavailable Presence:not_set Watchers 0
----------------
- 6 hints registered
```
В дополнение к отображению состояния каждого хинта, вывод `core show hints` также выводит количество наблюдателей. _Наблюдатель_ - это объект, который подписался на получение обновлений о состоянии этого расширения. Если конечная точка SIP подписывается на состояние добавочного номера, количество наблюдателей будет увеличено.

Состояние расширения также может быть получено с помощью функции диалплана `EXTENSION_STATE()`. Эта функция работает так же, как `DEVICE_STATE()`, описанная в предыдущем разделе. Добавьте следующий пример в файл _/etc/asterisk/extensions.conf_ как новое расширение сразу после 235:

```
exten => 234,1,NoOp()
  same => n,Set(FEATURE(parkingtime)=60)

exten => 235,1,Noop(The state of 100@hints is ${EXTENSION_STATE(100@hints)} )
  same => n,Hangup()

exten => 321,1,NoOp()
```
Когда это расширение вызывается из конечной точки с номером 100, в консоли Asterisk отобразится следующее сообщение:
```
 -- The state of 100@hints is INUSE
```
В следующем списке перечислены возможные значения, которые могут быть возвращены функцией `EXTENSION_STATE()`:

* `UNKNOWN`
* `NOT_INUSE`
* `INUSE`
* `BUSY`
* `UNAVAILABLE`
* `RINGING`
* `RINGINUSE`
* `HOLDINUSE`
* `ONHOLD`

## SIP Presence

Asterisk gives devices the capability to subscribe to extension state using the SIP protocol. This functionality is often referred to as BLF \(Busy Lamp Field\); see [Figure 13-1](13.%20Device%20States%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22fig1301).

![](.gitbook/assets/0%20%282%29.png)

#### Figure 13-1. Busy Lamp Field aka sidecar

The configuration of the module will be slightly \(or very\) different for each manufacturer; however, the subscription information will—one way or another—need to include the following:

* The address of the Asterisk server \(this might be defined on a per-button basis, or it might apply to the whole phone\).
* The context to subscribe to \(in our sample dialplan, it’s named \[hints\]\). This setting is defined in the subscribe\_context field of the asterisk.ps\_endpoints table.
* The relevant extension \(100, 101, 102, etc.\)[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405538488)

One of the more simple and inexpensive ways we’ve found for testing presence is using the open source Windows softphone, MicroSIP.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405536680) You’ll first need to download MicroSIP and get it registered to your Asterisk system. Then, under the contacts tab of the softphone, you can right-click in the open area to Add a contact. In the Name section you can put whatever you wish, but under the Number section, you will input extension@hints context, which in our case would be one of 100@hints, 101@hints, 102@hints, or 103@hints. If you’ve set everything up in Asterisk per the previous examples, you should see the state of your subscriptions change in response to whatever the far end set is doing. You can also monitor this from Asterisk’s perspective using a command such as:

$ watch -n 0.5 "sudo asterisk -rx 'core show hints'"

The configuration of presence on physical desk telephones is essentially the same, but it can be more difficult to make sense of the specific syntax each manufacturer requires. Our advice is to get it working with MicroSIP \(which you should be able to run on WINE under Linux or macOS\). It’s an easy setup, and from there you’ll have a known-good configuration you can trust when you’re sorting out a similar config for one of your desk phones.

## Использование пользовательских состояний устройств

In addition to the devices Asterisk knows internally how to monitor (PJSIP, ConfBridge, Park, Calendar), Asterisk also provides the ability to create custom device states, which can be very useful in the development of some interesting applications.

Custom device states are defined using a prefix of Custom:. The text that comes after the prefix can be anything you want. To set or read the value of a custom device state, use the DEVICE_STATE() dialplan function. Put this into your extensions.conf right after extension 235:
```
exten => 235,1,Noop(The state of 100@hints is ${EXTENSION_STATE(100@hints)} )
  same => n,Hangup()

exten => 236,1,Noop(Set a custom status)
  same => n(blink),Set(DEVICE_STATE(Custom:rudolph)=UNAVAILABLE)
  same => n,Set(DEVICE_STATE(Custom:santa)=NOT_INUSE)
  same => n,Wait(0.75)
  same => n,Set(DEVICE_STATE(Custom:rudolph)=NOT_INUSE)
  same => n,Set(DEVICE_STATE(Custom:santa)=UNAVAILABLE)
  same => n,Wait(0.75)
  same => n,Goto(blink)
```
Then add this to your `[hints]` context:
```
exten => 221,hint,ConfBridge:221
exten => santa,hint,Custom:santa
exten => rudolph,hint,Custom:rudolph
```
Festive, yeah?

#### Note

You will notice that when you hang up, one of the custom device states will remain “Unavailable.” This is an important point: there is nothing in the system that will update your custom device states, unless you yourself have implemented something to do that.

## Conclusion

The device states functionality in Asterisk can be used to track the state of various resources and deliver information about those states to various subscribers. Commonly \(and traditionally\) used for Busy Lamp Fields, the Custom device state allows this resource to be far more flexible than it would be in a traditional PBX.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405538488-marker) Items 2 and 3 may be formed as a single string, looking like 100@hints, or something similar.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405536680-marker) Which is written using the same PJSIP library that Asterisk uses.
