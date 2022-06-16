# IPA-dhcp

:warning: **Don’t use this project it's still in development**









This is a rudimentary plugin that adds DHCP functionality to [FreeIPA](http://www.freeipa.org).

Это элементарный плагин, который добавляет функциональность DHCP к [FreeIPA] (http://www.freeipa.org).

This plugin can be used in one of two ways. If you want, you can install [ISC DHCP](https://www.isc.org/downloads/dhcp/) on your FreeIPA server itself and let your FreeIPA server act as a DHCP server; this approximately mirrors the way FreeIPA can run ISC BIND and act as a DNS server. Or if you prefer you can run ISC DHCP on another, separate server and point it at your FreeIPA server via an anonymous LDAP binding. Both methods are really exactly the same; the only difference is where you install and run the ISC DHCP software.

Этот плагин можно использовать одним из двух способов. Если вы хотите, вы можете установить [ISC DHCP](https://www.isc.org/downloads/dhcp/) на свой сервер FreeIPA и позволить вашему серверу FreeIPA действовать как DHCP-сервер; это примерно отражает то, как FreeIPA может запускать ISC BIND и действовать как DNS-сервер. Или, если вы предпочитаете, вы можете запустить ISC DHCP на другом, отдельном сервере и указать его на свой сервер FreeIPA через анонимную привязку LDAP. Оба метода действительно одинаковы; единственная разница заключается в том, где вы устанавливаете и запускаете программное обеспечение ISC DHCP.
## Pictures

<center><a href="http://i.imgur.com/Q0hTeDu.png"><img width="200px" src="http://i.imgur.com/Q0hTeDu.png"></a> <a href="http://i.imgur.com/uczre41.png"><img width="200px" src="http://i.imgur.com/uczre41.png"></a> <a href="http://i.imgur.com/aNAkxwd.png"><img width="200px" img src="http://i.imgur.com/aNAkxwd.png"></a> <a href="http://i.imgur.com/ClkYgzy.png"><img width="200px" img src="http://i.imgur.com/ClkYgzy.png"></a> <a href="http://i.imgur.com/ysmKgXL.png"><img width="200px" img src="http://i.imgur.com/ysmKgXL.png"></a> <a href="http://i.imgur.com/IALwjJU.png"><img width="200px" img src="http://i.imgur.com/IALwjJU.png"></a> <a href="http://i.imgur.com/HeqlaoQ.png"><img width="200px" img src="http://i.imgur.com/HeqlaoQ.png"></a></center>

## An important caveat

This plugin was built to purpose. It's not a totally general-purpose solution for DHCP integration into FreeIPA. Some major features of ISC DHCP are currently not supported at all by this plugin, including shared networks, classes and DHCP failover. It's not that those features _can't_ be supported; it's just that I don't personally need them right now, so I haven't added them. So it might be better to think of this plugin as a sort of proof of concept, or maybe a reference implementation of the DHCP schema, rather than a piece of finished software for general use.

That being said, you, Constant Reader, are welcome to this software. If you can use it as is, great. If not but it's useful to you as a springboard toward your own solution, also great. Either way, welcome and good luck.

Этот плагин был создан специально. Это не совсем универсальное решение для интеграции DHCP в FreeIPA. Некоторые основные функции ISC DHCP в настоящее время вообще не поддерживаются этим подключаемым модулем, включая общие сети, классы и отказоустойчивость DHCP. Дело не в том, что эти функции не поддерживаются; просто мне лично они сейчас не нужны, поэтому я их не добавлял. Так что, возможно, лучше рассматривать этот плагин как своего рода доказательство концепции или, возможно, эталонную реализацию схемы DHCP, а не часть готового программного обеспечения для общего использования.

При этом вы, постоянный читатель, добро пожаловать в это программное обеспечение. Если вы можете использовать его как есть, отлично. Если нет, но это полезно для вас как трамплин для вашего собственного решения, тоже отлично. В любом случае, добро пожаловать и удачи.


## How it works in a nutshell

The DHCP LDAP schema is, let's say, not necessarily the easiest thing in the world to work with, and its implementation in ISC DHCP is incomplete. So I've chosen to take a fairly minimalist approach to this plugin.

This plugin supports a single, global configuration rooted at `cn=dhcp,$SUFFIX`. That's the `dhcpService` object. It has a handful of global attributes, and it also has links to the `dhcpServer` objects that identify the DHCP server hosts that get their configuration from this server.

Beneath `cn=dhcp` there are `dhcpSubnet` objects, one for each — surprise — subnet the DHCP server services. If the server receives a DHCP request from a device on a particular subnet, it will respond to that request if and only if there's a `dhcpSubnet` object in the tree that corresponds to that subnet. If there's no `dhcpSubnet` object, the server will ignore all requests from that subnet.

Each `dhcpSubnet` may have one or more `dhcpPool` objects under it. A `dhcpPool` has a range attribute that sets the first and last IP address in the pool; the DHCP server will draw from the IPs in that range when handing out dynamic leases.

And then there's the labor-saving part, the part which I personally find most useful but which I can imagine most people will find least appealing: Any "host" in the IPA database which has both a MAC address and an A record gets a `dhcpHost` object generated for it automatically and dynamically. This is _why_ I wrote this plugin in the first place. In my environment, I want every device in the system to be able to get its IP address via DHCP (as long as it's plugged into the correct switch port). Most people probably don't want that, or at least not _exactly_ that. But that's what I want, so that's what I wrote.

It would, of course, be possible to change the plugin such that `dhcpHost` objects aren't dynamic but rather are generated statically, giving the user the option of creating a `dhcpHost` from an existing host record, then making the `dhcpHost` records editable in the UI in much the same way that DNS records already are. That would be, as the saying goes, a "simple matter of programming."

Схема DHCP LDAP, скажем так, не обязательно самая простая в мире для работы, и ее реализация в ISC DHCP неполна. Поэтому я выбрал довольно минималистский подход к этому плагину.

Этот плагин поддерживает единую глобальную конфигурацию с корнем `cn=dhcp,$SUFFIX`. Это объект `dhcpService`. У него есть несколько глобальных атрибутов, а также ссылки на объекты dhcpServer, которые идентифицируют хосты DHCP-сервера, которые получают свою конфигурацию с этого сервера.

Под `cn=dhcp` находятся объекты `dhcpSubnet`, по одному для каждой (сюрприз) подсети службы DHCP-сервера. Если сервер получает запрос DHCP от устройства в определенной подсети, он ответит на этот запрос тогда и только тогда, когда в дереве есть объект `dhcpSubnet`, соответствующий этой подсети. Если объекта dhcpSubnet нет, сервер будет игнорировать все запросы из этой подсети.

Под каждым dhcpSubnet может находиться один или несколько объектов dhcpPool. `dhcpPool` имеет атрибут диапазона, который устанавливает первый и последний IP-адрес в пуле; DHCP-сервер будет использовать IP-адреса в этом диапазоне при динамической аренде.

А затем есть трудосберегающая часть, часть, которую я лично нахожу наиболее полезной, но которая, я могу представить, покажется наименее привлекательной для большинства людей: любой «хост» в базе данных IPA, который имеет как MAC-адрес, так и запись A, получает ` dhcpHost`, созданный для него автоматически и динамически. Это _почему_ я написал этот плагин в первую очередь. В моей среде я хочу, чтобы каждое устройство в системе могло получить свой IP-адрес через DHCP (при условии, что оно подключено к правильному порту коммутатора). Большинство людей, вероятно, этого не хотят, или, по крайней мере, не совсем этого. Но это то, что я хочу, поэтому я написал.

Конечно, можно было бы изменить плагин таким образом, чтобы объекты dhcpHost не были динамическими, а генерировались статически, давая пользователю возможность создать dhcpHost из существующей записи хоста, а затем создать dhcpHost. ` записи, редактируемые в пользовательском интерфейсе почти так же, как записи DNS. Это было бы, как говорится, «простым вопросом программирования».

## How to use it

Once you've configured the plugin how you like, either via the IPA command line (start with `ipa help dhcp` and go from there) or via the web GUI, you need to configure an ISC DHCP server to talk to it. Install the server software by doing a 

После того, как вы настроили плагин так, как вам нравится, либо через командную строку IPA (начните с `ipa help dhcp` и продолжайте оттуда), либо через веб-интерфейс, вам нужно настроить DHCP-сервер ISC для связи с ним. Установите серверное программное обеспечение, выполнив

```
yum install -y dhcp or dnf install -y dhcp-server
```

to get what you need (assuming you're using Red Hat Enterprise Linux or CentOS; details will differ if you're on Fedora). Once the server is installed, edit the file `/etc/dhcp/dhcpd.conf` and make it look like this:

чтобы получить то, что вам нужно (при условии, что вы используете Red Hat Enterprise Linux или CentOS; детали будут отличаться, если вы используете Fedora либо другие дистрибутивы). После установки сервера отредактируйте файл `/etc/dhcp/dhcpd.conf` и придайте ему следующий вид:

```
ldap-server "ipa.example.com";
ldap-port 389;
ldap-base-dn "cn=dhcp,dc=example,dc=com";
ldap-method static;
ldap-debug-file "/var/log/dhcp-ldap-startup.log";
```

Obviously the hostname `ipa.example.com` should instead be the hostname of your FreeIPA server, and the base DN suffix `dc=example,dc=com` should instead be your own base DN suffix. But other than that, make your config file look like this; this sets up an anonymous bind to the FreeIPA LDAP server.

Очевидно, что имя хоста `ipa.example.com` вместо этого должно быть именем хоста вашего сервера FreeIPA, а суффикс базового DN `dc=example,dc=com` вместо этого должен быть вашим собственным суффиксом базового DN. Но кроме этого, сделайте так, чтобы ваш конфигурационный файл выглядел следующим образом: это устанавливает анонимную привязку к серверу LDAP FreeIPA.

To test out the configuration, as root on your DHCP server, run the following command:

Чтобы проверить конфигурацию, от имени root на DHCP-сервере выполните следующую команду:

```
dhcpd -d
```

You should get something like this (with your own MAC and IP address, obviously):

Вы должны получить что-то вроде этого (очевидно, с вашим собственным MAC и IP-адресом):

```
Internet Systems Consortium DHCP Server 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Wrote 0 deleted host decls to leases file.
Wrote 0 new dynamic host decls to leases file.
Wrote 0 leases to leases file.
Listening on LPF/eth0/00:50:56:1e:01:1f/10.30.1.0/24
Sending on   LPF/eth0/00:50:56:1e:01:1f/10.30.1.0/24
Sending on   Socket/fallback/fallback-net
```

If you get any error messages, check your DHCP configuration in FreeIPA and try again. Assuming DHCPd started up correctly, hit control-C to stop it and then check the contents of `/var/log/dhcp-ldap-startup.log`. Modulo some awkward line breaks, this file should look like a dhcpd.conf file, something like this:

Если вы получаете какие-либо сообщения об ошибках, проверьте конфигурацию DHCP в FreeIPA и повторите попытку. Предполагая, что DHCPd запустился правильно, нажмите Control-C, чтобы остановить его, а затем проверьте содержимое `/var/log/dhcp-ldap-startup.log`. По модулю некоторых неудобных разрывов строк этот файл должен выглядеть как файл dhcpd.conf, примерно так:

```
authoritative;
default-lease-time 43200;
max-lease-time 86400;
one-lease-per-client on;
option domain-name-servers ns.la.charlietango.com;
option domain-name "la.charlietango.com";
option domain-search "la.charlietango.com", "charlietango.com";

host mac1.la.charlietango.com {
    hardware ethernet 01:02:03:04:05:06;
    fixed-address mac1.la.charlietango.com;
    option host-name "mac1.la.charlietango.com";
}

subnet 10.30.1.0 netmask 255.255.255.0 {
    option broadcast-address 10.30.1.255;
    option subnet-mask 255.255.255.0;
    option routers 10.30.1.254;
    pool {
        range 10.30.1.201 10.30.1.249;
        allow known-clients;
        allow unknown-clients;
        max-lease-time 86400;
        default-lease-time 43200;
    }
}
```

I've fixed the line breaks and indentation to make it more readable, but other than that, that's what DHCPd generated for me based on the configuration I created in my FreeIPA server. Note that I have one host in FreeIPA that has a MAC address (the obviously bogus `01:02:03:04:05:06`) and an IP address; this was automatically turned into a global DHCP host record. Below that, you see my one DHCP subnet containing its single address pool.

Now, to get this config generated I set my DHCP server's `ldap-method` to `static`. This is great for testing but _not_ for production. See, when `ldap-method` is set to `static` the DHCP server queries the LDAP server _once_ at start-up, downloads the whole tree, converts that tree into a running configuration and then _never talks to the LDAP server again_ until DHCPd is restarted. This stinks because you have to restart DHCPd every time you add a host to the realm! It's better by far to set `ldap-method` to `dynamic`. This tells DHCPd to read its _static_ configuration once at startup, but whenever a request comes in for a lease to query the LDAP server anew. Changes to the LDAP configuration, then, can propagate to the DHCP server(s) without having to restart them.

Я исправил разрывы строк и отступы, чтобы сделать его более читабельным, но в остальном это то, что DHCPd сгенерировал для меня на основе конфигурации, которую я создал на своем сервере FreeIPA. Обратите внимание, что у меня есть один хост в FreeIPA, у которого есть MAC-адрес (очевидно поддельный `01:02:03:04:05:06`) и IP-адрес; это было автоматически преобразовано в глобальную запись хоста DHCP. Ниже вы видите мою единственную подсеть DHCP, содержащую единственный пул адресов.

Теперь, чтобы сгенерировать эту конфигурацию, я установил `ldap-method` моего DHCP-сервера на `static`. Это отлично подходит для тестирования, но не для производства. Видите ли, когда для `ldap-method` установлено значение `static`, сервер DHCP запрашивает сервер LDAP _один раз_ при запуске, загружает все дерево, преобразует это дерево в рабочую конфигурацию, а затем _никогда больше не обращается к серверу LDAP_ до тех пор, пока DHCPd перезапускается. Это воняет, потому что вам приходится перезапускать DHCPd каждый раз, когда вы добавляете хост в область! Гораздо лучше установить «ldap-method» в «динамический». Это говорит DHCPd прочитать свою _статическую_ конфигурацию один раз при запуске, но всякий раз, когда поступает запрос на аренду, запрашивать сервер LDAP заново. Таким образом, изменения в конфигурации LDAP могут распространяться на сервер(ы) DHCP без необходимости их перезапуска.

So for production, make your `dhcpd.conf` look like this:
Итак, для производства сделайте ваш `dhcpd.conf` таким:
```
ldap-server "ipa.example.com";
ldap-port 389;
ldap-base-dn "cn=dhcp,dc=example,dc=com";
ldap-method dynamic;
ldap-debug-file "/var/log/dhcp-ldap-startup.log";
```

The `ldap-debug-file` line remains optional. If you leave it in and start up DHCPd, this time you'll get a config that looks like this:

```
authoritative;
default-lease-time 43200;
max-lease-time 86400;
one-lease-per-client on;
option domain-name-servers ns.la.charlietango.com;
option domain-name "la.charlietango.com";
option domain-search "la.charlietango.com", "charlietango.com";

subnet 10.30.1.0 netmask 255.255.255.0 {
    option broadcast-address 10.30.1.255;
    option subnet-mask 255.255.255.0;
    option routers 10.30.1.254;
    pool {
        range 10.30.1.201 10.30.1.249;
        allow known-clients;
        allow unknown-clients;
        max-lease-time 86400;
        default-lease-time 43200;
    }
}
```

Notice that this looks just the same as before, only the host entry isn't there. DHCPd will query the LDAP server for `dhcpHost` objects every time a DHCP request comes in … which, if you have a big, busy network with a lot of DHCP requests, can put some load on your LDAP server. But this can be addressed with replication, so it's rarely a real issue.
Обратите внимание, что это выглядит точно так же, как и раньше, только здесь нет записи хоста. DHCPd будет запрашивать у LDAP-сервера объекты dhcpHost каждый раз, когда приходит DHCP-запрос… , что, если у вас большая загруженная сеть с большим количеством DHCP-запросов, может создать некоторую нагрузку на ваш LDAP-сервер. Но это можно решить с помощью репликации, так что это редко бывает реальной проблемой.

## Areas for improvement

There are some pretty obvious low-hanging fruit that I haven't bothered to pluck.
Есть несколько довольно очевидных низко висящих плодов, которые я не удосужился сорвать.

### Shared networks

ISC DHCP includes this concept called a "shared network," which is a topology in which multiple disjoint IP networks exist on the same physical network — where "physical network" here includes _logical_ networks like VLANs. It's a pretty rarefied idea, really. It boils down to the idea that on a single broadcast domain you might have devices in the 172.16.1.0/24 network _and also_ devices in the 172.16.2.0/24 network … again, _on a single broadcast domain._ Not on separate segments connected by a router, not on separate sets of ports belonging to different VLANs, but _all on the same switch_ for some reason. I'm sure there are people out there who use this kind of topology and probably for good reason, but in all my years I've never actually seen it deployed, so I've not bothered to add support for it into this plugin.
ISC DHCP включает в себя эту концепцию, называемую «общая сеть», которая представляет собой топологию, в которой несколько непересекающихся IP-сетей существуют в одной и той же физической сети, где «физическая сеть» здесь включает _логические_ сети, такие как VLAN. На самом деле это довольно редкая идея. Это сводится к идее, что в одном широковещательном домене у вас могут быть устройства в сети 172.16.1.0/24 _а также_ устройства в сети 172.16.2.0/24… опять же, _в одном широковещательном домене._ Не на отдельных сегментах, связанных маршрутизатором, а не на отдельных наборах портов, принадлежащих разным VLAN, а по какой-то причине _всех на одном коммутаторе_. Я уверен, что есть люди, которые используют такую топологию и, вероятно, не без оснований, но за все мои годы я ни разу не видел ее развернутой, поэтому я не удосужился добавить ее поддержку в этот плагин.

### Groups and classes

Groups and classes are labor-saving devices in the DHCP config file, basically. They're data structures that let you assign different parameters to hosts automatically rather than having to create a large, complex config file with a lot of copying and pasting. You can put a set of hosts under a group in order to give them some common options, or you can set up a class so hosts sharing common characteristics get the right options automatically.

I'm not using groups or classes, so I haven't added any support for them. It's really that simple.

По сути, группы и классы — это устройства для экономии труда в конфигурационном файле DHCP. Это структуры данных, которые позволяют автоматически назначать хостам различные параметры, вместо того чтобы создавать большой и сложный файл конфигурации с большим количеством операций копирования и вставки. Вы можете поместить набор хостов в группу, чтобы предоставить им некоторые общие параметры, или вы можете настроить класс, чтобы хосты с общими характеристиками автоматически получали правильные параметры.

Я не использую группы или классы, поэтому я не добавлял для них никакой поддержки. Это действительно так просто.

### Failover

If I'm not mistaken, there's _no_ special LDAP support for DHCP failover in ISC DHCP 4.2.5, meaning it would all have to be configured using `dhcpStatements` and such. I haven't taken the time to do this, though I probably will eventually.

Если я не ошибаюсь, в ISC DHCP 4.2.5 нет специальной поддержки LDAP для аварийного переключения DHCP, а это означает, что все это должно быть настроено с использованием `dhcpStatements` и тому подобного. Я не нашел времени, чтобы сделать это, хотя я, вероятно, в конечном итоге.
