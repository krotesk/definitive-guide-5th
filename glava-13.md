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

## Extension States Using the hint Directive

Extension state is a dialplan mechanism Asterisk uses to allow SIP devices to subscribe to presence information. As an example, a reception phone might have a Busy Lamp Field \(BLF\) module, containing buttons to be used to show the state of various phones in the office. The phone with the BLF will send subscription requests in order to tell Asterisk which devices it wants to receive presence information from. In the dialplan, we use the hint directive to define the mapping between an extension and one or more devices.

### Hints

To define a hint in the dialplan, the keyword hint is used in place of a priority.

\[hints\]

;exten = &lt;extension&gt;,hint,&lt;device state id&gt;\[& &lt;more dev state id\],&lt;presence state id&gt;

exten =&gt; 100,hint,${UserA\_DeskPhone}

exten =&gt; 221,hint,ConfBridge:221

Often, you might see hints defined in the same section of the dialplan as the normal extension. This can get a bit visually cluttered, and it also suggests that the hint is somehow associated with the dialable extension, which is not the case.

\[sets\]

exten =&gt; 100,hint,${UserA\_DeskPhone}

exten =&gt; 100,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_DeskPhone},${EXTEN},default,22\)\)

exten =&gt; 101,hint,${UserA\_SoftPhone}

exten =&gt; 101,1,Gosub\(subDialUser,${EXTEN},1\(${UserA\_SoftPhone},${EXTEN},default,23\)\)

exten =&gt; 102,hint,${UserB\_DeskPhone}

exten =&gt; 102,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_DeskPhone},${EXTEN},default,26\)\)

exten =&gt; 103,hint,${UserB\_SoftPhone}

exten =&gt; 103,1,Gosub\(subDialUser,${EXTEN},1\(${UserB\_SoftPhone},${EXTEN},default,24\)\)

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

In our example we’ve made a direct correlation between the hint’s extension number and the extension number being dialed, although there is no requirement that this be the case.

### Checking Extension States

The easiest way to check the current state of the hint extensions is through the Asterisk CLI. The core show hints command will display all currently configured hints:

\*CLI&gt; core show hints

 -= Registered Asterisk Dial Plan Hints =-

100@hints : PJSIP/0000f30A0A01 State:Unavailable Presence:not\_set Watchers 0

101@hints : PJSIP/SOFTPHONE\_A State:Unavailable Presence:not\_set Watchers 0

102@hints : PJSIP/0000f30B0B02 State:Unavailable Presence:not\_set Watchers 0

103@hints : PJSIP/SOFTPHONE\_B State:Unavailable Presence:not\_set Watchers 0

110@hints : PJSIP/0000f30A0A01&P State:Unavailable Presence:not\_set Watchers 0

221@hints : ConfBridge:221 State:Unavailable Presence:not\_set Watchers 0

----------------

- 6 hints registered

In addition to showing you the state of each hint, the output of core show hints also provides a count of watchers. A watcher is an entity that has subscribed to receive updates on the state of this extension. If a SIP endpoint subscribes to the state of an extension, the watcher count will be increased.

Extension state can also be retrieved with a dialplan function, EXTENSION\_STATE\(\). This function operates much like the DEVICE\_STATE\(\) function described in the preceding section. Add the following example to your /etc/asterisk/extensions.conf file, as a new extension right after 235:

exten =&gt; 234,1,NoOp\(\)

 same =&gt; n,Set\(FEATURE\(parkingtime\)=60\)

exten =&gt; 235,1,Noop\(The state of 100@hints is ${EXTENSION\_STATE\(100@hints\)} \)

 same =&gt; n,Hangup\(\)

exten =&gt; 321,1,NoOp\(\)

When this extension is called from the endpoint assigned to 100, this is the message that shows up on the Asterisk console:

 -- The state of 100@hints is INUSE

The following list includes the possible values that may be returned from the EXTENSION\_STATE\(\) function:

* UNKNOWN
* NOT\_INUSE
* INUSE
* BUSY
* UNAVAILABLE
* RINGING
* RINGINUSE
* HOLDINUSE
* ONHOLD

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

In addition to the devices Asterisk knows internally how to monitor \(PJSIP, ConfBridge, Park, Calendar\), Asterisk also provides the ability to create custom device states, which can be very useful in the development of some interesting applications.

Custom device states are defined using a prefix of Custom:. The text that comes after the prefix can be anything you want. To set or read the value of a custom device state, use the DEVICE\_STATE\(\) dialplan function. Put this into your extensions.conf right after extension 235:

exten =&gt; 235,1,Noop\(The state of 100@hints is ${EXTENSION\_STATE\(100@hints\)} \)

 same =&gt; n,Hangup\(\)

exten =&gt; 236,1,Noop\(Set a custom status\)

 same =&gt; n\(blink\),Set\(DEVICE\_STATE\(Custom:rudolph\)=UNAVAILABLE\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:santa\)=NOT\_INUSE\)

 same =&gt; n,Wait\(0.75\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:rudolph\)=NOT\_INUSE\)

 same =&gt; n,Set\(DEVICE\_STATE\(Custom:santa\)=UNAVAILABLE\)

 same =&gt; n,Wait\(0.75\)

 same =&gt; n,Goto\(blink\)

Then add this to your \[hints\] context:

exten =&gt; 221,hint,ConfBridge:221

exten =&gt; santa,hint,Custom:santa

exten =&gt; rudolph,hint,Custom:rudolph

Festive, yeah?

#### Note

You will notice that when you hang up, one of the custom device states will remain “Unavailable.” This is an important point: there is nothing in the system that will update your custom device states, unless you yourself have implemented something to do that.

## Conclusion

The device states functionality in Asterisk can be used to track the state of various resources and deliver information about those states to various subscribers. Commonly \(and traditionally\) used for Busy Lamp Fields, the Custom device state allows this resource to be far more flexible than it would be in a traditional PBX.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405538488-marker) Items 2 and 3 may be formed as a single string, looking like 100@hints, or something similar.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22idm46178405536680-marker) Which is written using the same PJSIP library that Asterisk uses.
