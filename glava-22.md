---
description: Безопасность
---

# Глава 22

> _Мы тратим время на поиски безопасности и ненавидим, когда получаем ее._ 
>
> -- Джон Стейнбек

Безопасность вашей системы Asterisk имеет решающее значение, особенно если система подключена к Интернету. Злоумышленники могут заработать много денег, используя системы для бесплатных телефонных звонков. В этой главе даются советы о том, как обеспечить более надежную защиту для вашего развертывания VoIP.

## Сканирование действительных учетных записей

Если вы выставляете свою систему Asterisk в общедоступный Интернет, одна из вещей, которую вы почти наверняка увидите - это сканирование действительных учетных записей. Пример 22-1 содержит записи лога с одной из производственных систем Asterisk авторов.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396108536) Это сканирование началось с проверки различных общих имен пользователей, а затем перешло к сканированию числовых аккаунтов. Обычно пользователи называют учетные записи SIP так же, как и внутренние номера на АТС. Это сканирование использует этот факт.

{% hint style="info" %}
**Подсказка**

Используйте нечисловые имена пользователей для своих учетных записей VoIP, чтобы их было сложнее угадать. Например, в этой книге мы используем MAC-адрес SIP-телефона в качестве имени учетной записи в Asterisk.
{% endhint %}

#### Пример 22-1. Отрывок лога сканирования учетной записи

```text
[Aug 22 15:17:15] NOTICE[25690] chan_sip.c: Registration from
'"123"<sip:123@127.0.0.1>' failed for '203.86.167.220:5061' - No matching peer
found
[Aug 22 15:17:15] NOTICE[25690] chan_sip.c: Registration from
'"1234"<sip:1234@127.0.0.1>' failed for '203.86.167.220:5061' - No matching peer
found
[Aug 22 15:17:15] NOTICE[25690] chan_sip.c: Registration from
'"12345"<sip:12345@127.0.0.1>' failed for '203.86.167.220:5061' - No matching peer
found
...
[Aug 22 15:17:17] NOTICE[25690] chan_sip.c: Registration from
'"100"<sip:100@127.0.0.1>' failed for '203.86.167.220:5061' - No matching peer found
[Aug 22 15:17:17] NOTICE[25690] chan_sip.c: Registration from
'"101"<sip:101@127.0.0.1>' failed for '203.86.167.220:5061' - No matching peer found
```

В любой системе логи будут полны попыток вторжения. Это просто характер подключения систем к интернету. В этой главе мы обсудим некоторые способы настройки вашей системы таким образом, чтобы она имела надежные механизмы для решения этих проблем.

## Уязвимости аутентификации

В первом разделе этой главы обсуждалось сканирование имен пользователей. Даже если у вас есть имена пользователей, которые трудно угадать, очень важно, чтобы у вас были надежные пароли. Если злоумышленник может получить действительное имя пользователя, он, скорее всего, попытается подобрать пароль. Надежные пароли делают это намного сложнее.

Схема аутентификации по умолчанию SIP-протокола является слабой. Аутентификация выполняется с помощью механизма вызова и ответа MD5. Если злоумышленник может перехватить любой трафик, например SIP-вызов, выполненный с ноутбука в открытой беспроводной сети, ему будет намного проще работать с брутфорсом паролей, поскольку это не потребует запросов аутентификации у сервера.

{% hint style="info" %}
**Подсказка**

Используйте надежные пароли. В Интернете доступно множество ресурсов, которые помогают определить, что представляет собой надежный пароль. Есть также много надежных генераторов паролей. Используй их!
{% endhint %}

## Fail2ban

В предыдущих двух разделах рассматривались атаки, связанные со сканированием действительных имен пользователей и подбором паролей брутфорсом. [Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) - это приложение, которое может просматривать журналы Asterisk и обновлять правила брандмауэра, чтобы блокировать источник атаки в ответ на слишком большое количество неудачных попыток аутентификации.

{% hint style="info" %}
**Подсказка**

Используйте Fail2ban при предоставлении услуг Voice over IP в ненадежных сетях. Оно автоматически обновит правила брандмауэра, чтобы заблокировать источники атак.
{% endhint %}

### Установка

Fail2ban доступен в виде пакета во многих дистрибутивах. Кроме того, вы можете установить его из исходников, загрузив с веб-сайта Fail2ban. Чтобы установить Fail2ban на RHEL, необходимо включить репозиторий EPEL \(который был рассмотрен в [Главе 3](glava-03.md)\). Вы можете установить Fail2ban, выполнив следующую команду:

```text
$ sudo yum install fail2ban
```

{% hint style="info" %}
**Примечание**

Установка Fail2ban из пакета будет включать скрипт запуска, чтобы гарантировать запуск при загрузке компьютера. Если вы устанавливаете из исходных кодов, убедитесь, что вы предпринимаете необходимые шаги для гарантированной постоянной работы Fail2ban.
{% endhint %}

### Конфигурация

Во-первых, мы хотим настроить журнал безопасности в Asterisk, который Fail2ban может использовать.

```text
$ sudo vim /etc/asterisk/logger.conf
```

Раскомментировать \(или добавить\) строку, которая разрешает чтение `security => security` и измените `dateformat` даты для понимания её в журнале fail2ban.

```text
[general]
exec_after_rotate=gzip -9 ${filename}.2;
dateformat = %F %T
[logfiles]
;debug => debug
security => security
;console => notice,warning,error,verbose
console => notice,warning,error,debug
messages => notice,warning,error
full => notice,warning,error,debug,verbose,dtmf,fax
```

Затем перезагрузите logger Asterisk:

```text
$ sudo asterisk -rx 'logger reload'
```

Поскольку текущие версии Fail2ban уже поставляются с определением изолятора Asterisk, все, что нам нужно сделать, это включить его:

Для этого рекомендуется создать файл _/etc/fail2ban/jail.local_ \(технически вы можете поместить его в _/etc/fail2ban/jail.conf_, но он скорее всего будет перезаписан\):

```text
$ sudo vim /etc/fail2ban/jail.local

[asterisk]
enabled  = true
filter   = asterisk
action   = iptables-allports[name=ASTERISK, protocol=all]
          sendmail[name=ASTERISK, dest=me@shifteight.org, sender=fail2ban@shifteight.org]
logpath  = /var/log/asterisk/messages
          /var/log/asterisk/security
maxretry = 5
findtime = 21600
bantime  = 86400
```

Мы установили запрет на 24 часа, но вы можете сделать время больше или меньше, как пожелаете \(время запрета определяется в секундах, так что его необходимо рассчитать\). Поскольку большинство атакующих хостов меняются через несколько часов, нет никакого вреда в разблокировании IP-адреса через 24 часа. Если хост атакует снова, он снова будет заблокирован.

О, вы также можете указать ему игнорировать ваш IP \(или любые другие IP-адреса, с которых можно получать попытки подключения\). Если вы случайно заблокировали себя, когда делали какую-то лабораторную работу и неправильно регистрировались, не волнуйтесь, вы в конечном итоге сделаете это с собой \(если, конечно, не создадите список игнорирования для соответствующих IP-адресов\).

```text
[DEFAULT]
ignoreip = <ip-адрес(а), разделенные запятыми>
[asterisk]
enabled = true
filter = asterisk
action = iptables-allports[name=ASTERISK, protocol=all]
 sendmail[name=ASTERISK, dest=me@shifteight.org, sender=fail2ban@shifteight.org]
logpath = /var/log/asterisk/messages
 /var/log/asterisk/security
maxretry = 5
findtime = 21600
bantime = 86400
```

Перезапустите Fail2ban и все будет хорошо.

```text
$ sudo systemctl reload fail2ban
```

Проверьте это, если можете, с IP-адреса, который вы не против заблокировать \(например, дополнительный компьютер в вашей лаборатории, который может стать объектом тестирования в данном случае\). Попытайтесь зарегистрироваться с использованием неверных учетных данных, и после пяти попыток \(или любого другого значения, для которого вы установили `maxretry`\) этот IP-адрес должен быть заблокирован.

Вы можете увидеть, какие адреса блокирует Asterisk jail, с помощью команды:

```text
$ sudo fail2ban-client status asterisk
```

И если вы хотите разблокировать IP,[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html#idm46178396052456) следующая команда должна сделать это.

```text
$ sudo fail2ban-client set asterisk unbanip <ip для разбанивания>
```

Дополнительную информацию о Fail2ban можно найти на странице [Fail2ban wiki](http://www.fail2ban.org/wiki/index.php/Main_Page).

## Шифрование медиапотока

В то время как мы приводили в этой книге примеры, которые использовали шифрование, имейте в виду, что вы можете настроить SIP так, что медиапоток будет отправляться в незашифрованном виде. В этом случае любой, кто перехватит RTP-трафик между двумя SIP-узлами, сможет использовать довольно простые инструменты для извлечения звука из этих вызовов.

## Dialplan Vulnerabilities

The Asterisk dialplan is another area where taking security into consideration is critical. The dialplan can be broken down into multiple contexts to provide access control to extensions. For example, you may want to allow your office phones to make calls out through your service provider. However, you do not want to allow anonymous callers that come into your main company menu to be able to then dial out through your service provider. Use contexts to ensure that only the callers you intend have access to services that cost you money.

{% hint style="info" %}
**Tip**

Build dialplan contexts with great care. Also, avoid putting any extensions that could cost you money in the \[default\] context.
{% endhint %}

One of the more recent Asterisk dialplan vulnerabilities to have been discovered and published is the idea of dialplan injection. A dialplan injection vulnerability begins with an extension that has a pattern that ends with the match-all character, a period. Take this extension as an example:

```text
exten => _X.,1,Dial(PJSIP/otherserver/${EXTEN},30)
```

The pattern for this extension matches all extensions \(of any length\) that begin with a digit. Patterns like this are pretty common and convenient. The extension then sends this call over to another server using the IAX2 protocol, with a dial timeout of 30 seconds. Note the usage of the ${EXTEN} variable here. That’s where the vulnerability exists.

In the world of Voice over IP, there is no reason that a dialed extension must be numeric. In fact, it is quite common using SIP to be able to dial someone by name. Since it is possible for non-numeric characters to be a part of a dialed extension, what would happen if someone sent a call to this extension?

```text
1234&DAHDI/g1/12565551212
```

A call like this is an attempt at exploiting a dialplan injection vulnerability. In the previous extension definition, once ${EXTEN} has been evaluated, the actual Dial\(\) statement that will be executed is:

```text
exten => _X.,1,Dial(PJSIP/otherserver/1234&DAHDI/g1/12565551212,30)
```

If the system has a PRI configured, this call will cause a call to go out on the PRI to a number chosen by the attacker, even though you did not explicitly grant access to the PRI to that caller. This problem can quickly cost you a whole lot of money.

There are several approaches to avoiding this problem. The first and easiest approach is to always use strict pattern matching. If you know the length of extensions you are expecting and expect only numeric extensions, use a strict numeric pattern match. For example, this would work if you are expecting four-digit numeric extensions only:

```text
exten => _XXXX,1,Dial(PJSIP/otherserver/${EXTEN},30)
```



Another approach to mitigating dialplan injection vulnerabilities is by using the FILTER\(\) dialplan function. Perhaps you would like to allow numeric extensions of any length. FILTER\(\) makes that easy to achieve safely:

```text
exten => _X.,1,Set(SAFE_EXTEN=${FILTER(0-9A-F,${EXTEN})})
 same => n,Dial(PJSIP/otherserver/${SAFE_EXTEN},30)
```

For more information about the syntax for the FILTER\(\) dialplan function, see the output of the core show function FILTER command at the Asterisk CLI.

A more comprehensive \(but also complex\) approach might be to have all dialed digits validated by functions outside of your dialplan \(for example, database queries that validate the dialed string against user permissions, routing patterns, restriction tables, and so forth\). This is a powerful concept, but beyond the scope of this book.

{% hint style="info" %}
**Tip**

Be wary of dialplan injection vulnerabilities. Use strict pattern matching or use the FILTER\(\) dialplan function to avoid these problems.
{% endhint %}

## Securing Asterisk Network APIs

To secure AGI, AMI, and ARI, you will need to carefully consider the following recommended practices:

* Only allow connections directly to the API from localhost/127.0.0.1.
* Use an appropriate framework in between the Asterisk API and your client application, and handle connection security through the framework.
* Control access to the framework and the system through strict firewall rules.

Beyond that, the same sort of security rules and best practices apply that you would follow in any mission-critical web application.

## Other Risk Mitigation

There are other useful features in Asterisk that can be used to mitigate the risk of attacks. The first is to use the permit and deny options to build access control lists \(ACLs\) for privileged accounts. Consider a PBX that has SIP phones on a local network, but also accepts SIP calls from the public internet. Calls coming in over the internet are only granted access to the main company menu, while local SIP phones have the ability to make outbound calls that cost you money. In this case, it is a very good idea to set ACLs to ensure that only devices on your local network can use the accounts for the phones.

In your ps\_endpoints table, the permit and deny options allow you to specify IP addresses, but you can also point to a label in the /etc/asterisk/acl.conf file. In fact, ACLs are accepted almost everywhere that connections to IP services are configured. For example, another useful place for ACLs is in /etc/asterisk/manager.conf, to restrict AMI accounts to the single host that is supposed to be using the manager interface.

ACLs can be defined in /etc/asterisk/acl.conf.

```text
[named_acl_1]
deny=0.0.0.0/0.0.0.0
permit=10.1.1.50
permit=10.1.1.55
[named_acl_2] ; Named ACLs support IPv6, as well.
deny=::
permit=::1/128
[local_phones]
deny=0.0.0.0/0.0.0.0
permit=192.168.0.0/255.255.0.0
```

Once named ACLs have been defined in acl.conf, have Asterisk load them using the reload acl command. Once loaded, they should be available via the Asterisk CLI:

```text
*CLI> module reload acl
*CLI> acl show
acl
---
named_acl_1
named_acl_2
local_phones
*CLI> acl show named_acl_1
ACL: named_acl_1
---------------------------------------------
 0: deny - 0.0.0.0/0.0.0.0
 1: allow - 10.1.1.50/255.255.255.255
 2: allow - 10.1.1.55/255.255.255.255
```

Now, instead of having to potentially repeat the same permit and deny entries in multiple places, you can apply an ACL by its name. You will find an acl field in the ps\_endpoints table, which you can use to point to a named ACL in the acl.conf file.

```text
mysql> select id,transport,aors,context,disallow,allow,acl from ps_endpoints;
|id |transport |aors |context|disallow|allow |acl |
|0000f30A0A01|transport-udp|0000f30A0A01|sets |all |ulaw |NULL|
|0000f30B0B02|transport-udp|0000f30B0B02|sets |all |ulaw |NULL|
|SOFTPHONE_A |transport-udp|SOFTPHONE_A |sets |all |ulaw,h264,vp8|NULL|
|SOFTPHONE_B |transport-udp|SOFTPHONE_B |sets |all |ulaw,h264,vp8|NULL|
mysql> update ps_endpoints
 set acl='local_phones'
 where id in ('0000f30A0A01','0000f30B0B02','SOFTPHONE_A','SOFTPHONE_B')
 ;
mysql> select id,transport,aors,context,disallow,allow,acl from ps_endpoints;
|id |transport |aors |context|disallow|allow |acl |
|0000f30A0A01|transport-udp|0000f30A0A01|sets |all |ulaw |local_phones|
|0000f30B0B02|transport-udp|0000f30B0B02|sets |all |ulaw |local_phones|
|SOFTPHONE_A |transport-udp|SOFTPHONE_A |sets |all |ulaw,h264,vp8|local_phones|
|SOFTPHONE_B |transport-udp|SOFTPHONE_B |sets |all |ulaw,h264,vp8|local_phones|
```

{% hint style="info" %}
**Tip**

Use ACLs when possible on all privileged accounts for network services.
{% endhint %}

Another way you can mitigate security risk is by configuring call limits. The recommended method for implementing call limits is to use the GROUP\(\) and GROUP\_COUNT\(\) dialplan functions. Here is an example that limits the number of calls from each SIP peer to no more than two at a time:

```text
exten => _X.,1,Set(GROUP(users)=${CHANNEL(endpoint)})
 same => n,NoOp(${CHANNEL(endpoint)} : ${GROUP_COUNT(${CHANNEL(endpoint)})} calls)
 same => n,GotoIf($[${GROUP_COUNT(${CHANNEL(endpoint)})} > 2]?denied:continue)
 same => n(denied),NoOp(There are too many calls up already. Hang up.)
 same => n,HangUp()
 same => n(continue),NoOp(continue processing call as normal here ...)
```

{% hint style="info" %}
**Tip**

Use call limits to ensure that if an account is compromised, it cannot be used to make hundreds of phone calls at a time.
{% endhint %}

## Resources

Some security vulnerabilities require modifications to the Asterisk source code to resolve. When those issues are discovered, the Asterisk development team puts out new releases that contain only fixes for the security issues, to allow for quick and easy upgrades. When this occurs, the Asterisk development team also publishes a security advisory document that discusses the details of the vulnerability. We recommend that you subscribe to the [asterisk-announce mailing list](http://lists.digium.com/mailman/listinfo/asterisk-announce) to make sure that you know about these issues when they come up.

{% hint style="info" %}
**Tip**

Subscribe to the asterisk-announce list to stay up to date on Asterisk security vulnerabilities.
{% endhint %}

One of the most popular tools for SIP account scanning and password cracking is [SIPVicious](http://sipvicious.org/). We strongly encourage that you take a look at it and use it to audit your own systems. If your system is exposed to the internet, others will likely run SIPVicious against it, so make sure that you do that first.

## Conclusion—A Better Idiot

There is a maxim in the technology industry that states, “As soon as something is made idiot-proof, nature will invent a better idiot.” The point of this statement is that no development effort can be considered complete. There is always room for improvement.

When it comes to security, you must always bear in mind that the people who are looking to take advantage of your system are highly motivated. No matter how secure your system is, somebody will always be looking to crack it.

We’re not advocating paranoia, but we are suggesting that what we have written here is by no means the final word on VoIP security. While we have tried to be as comprehensive as we can be in this book, you must accept responsibility for the security of your system.

The criminals are working hard to find weaknesses and exploit them.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396108536-marker) The real IP address has been replaced with 127.0.0.1 in the log entries.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch22.html%22%20/l%20%22idm46178396052456-marker) For example, yourself, because you forgot to define ignoreip...

