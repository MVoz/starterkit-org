## Введение ##

Некоторые полезные советы по удаленной работе с платой. Предполагается, что на плате запущен SSH-сервер, а в качестве ОС используется OsSetup или образ виртуальной машины с ftp starterkit.ru с Debian Lenny.

## Использование ключей для беспарольной работы с SSH ##

Для того, чтобы каждый раз не вводить пароль, нужно сгенерировать пару закрытый+открытый ключ на хост-системе, скопировать на плату открытый ключ и добавить о нем информацию для пользователя, от чьего имени будем работать на плате (root). Нужно выполнить команду ssh-keygen -t rsa, на все вопросы можно отвечать нажатием [Enter](Enter.md)
```
ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sasa/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/sasa/.ssh/id_rsa.
Your public key has been saved in /home/sasa/.ssh/id_rsa.pub.
```
копируем открытый ключ на плату
```
scp ~/.ssh/id_rsa.pub root@192.168.0.136:/root
root@192.168.0.136's password: 123456
```
заходим на плату через SSH  и добавляем информацию о ключе
```
ssh root@192.168.0.136
root@192.168.0.136's password: 123456
mkdir ~/.ssh
cat id_rsa.pub > ~/.ssh/authorized_keys
exit
```

## Файловый менеджер Midnight Commander (mc) ##

Для начинающих пользователей Linux будет полезен этот файловый менеджер https://www.midnight-commander.org/, работающий в консольном режиме. Версия в Ubuntu 10.04 и Debian Lenny старая и содержит ряд проблем, которые сейчас уже исправлены (впрочем, нет гарантий, что не добавлены новые). Увлекаться им не стоит: консоль, зачастую, предоставляет более удобные механизмы для работы.

### Обновление бинарного пакета mc ###

**Debian Lenny** официально уже не поддерживается, но можно взять пакет тут: http://www.tataranovich.com/debian/pool/lenny/main/m/mc/. Установить можно командой (в зависимости от скачанного пакета)
```
dpkg -i mc_4.7.5.6-1_i386.deb 
```

**Ubuntu 10.04** Для обновления стабильной версии (4.7.5 на момент описания) удобней и правильней воспользоваться [PPA](https://help.launchpad.net/Packaging/PPA) - корректная версия пакета для используемой ветки дистрибутива выбирается автоматически
```
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install mc
```

mc позволяет работать с удаленными устройствами по сети через ssh-соединение, при этом, файловая система устройства представляется как локальная и позволяет копиррование/редактирование файлов средствами файлового менеджера. Чтобы
установить соединение с платой, наберите в командной строке mc:<br>

<pre><code>cd sh://root@192.168.0.136<br>
</code></pre>

Так же, это можно сделать через меню:<br>
<pre><code>Right -&gt; Shell link<br>
</code></pre>

<img src='http://wiki.starterkit-org.googlecode.com/git/images/Screenshot-mc2.png' />

Введите имя<i>пользователя</i>на<i>удаленной_системе@ip.адрес<br>
<pre><code>Shell link to machine -&gt; root@192.168.0.136<br>
</code></pre></i>

<img src='http://wiki.starterkit-org.googlecode.com/git/images/Screenshot-mc3.png' />

Если не используются ключи SSH, в командной строке внизу появится приглашение для ввода пароля<br>
<br>
<img src='http://wiki.starterkit-org.googlecode.com/git/images/Screenshot-mc4.png' />

введите пароль, например<br>
<pre><code>root@192.168.0.136's password: 123456<br>
</code></pre>

после ввода пароля вы увидите корневую ФС удаленного устройства<br>
<br>
<img src='http://wiki.starterkit-org.googlecode.com/git/images/Screenshot-mc5.png' />

файлы можно копирровать/удалять/редактировать, как будто они находятся на вашем компьютере.<br>
<br>
В дистрибутивах Debian/Ubuntu использован DE Gnome. Консоль Gnome пересекается по ряду горячих клавиш с mc. Для удобства работы с mc можно привести настройки Gnome terminal к такому виду:<br>
<br>
<img src='http://wiki.starterkit-org.googlecode.com/git/images/Screenshot-mc-gnome_terminal.png' />

<h2>Работа в консоли через SSH</h2>

Мы можем выполнять удаленно команды на плате из терминала на хост-системе. Например, посмотрим информацию о процессоре, выполнив в терминале на хост-системе<br>
<pre><code>ssh root@192.168.0.136 cat /proc/cpuinfo<br>
Processor	: ARM926EJ-S rev 5 (v5l)<br>
BogoMIPS	: 199.06<br>
Features	: swp half thumb fastmult edsp java <br>
CPU implementer	: 0x41<br>
CPU architecture: 5TEJ<br>
CPU variant	: 0x0<br>
CPU part	: 0x926<br>
CPU revision	: 5<br>
<br>
Hardware	: Atmel AT91SAM9M10G45-EK<br>
Revision	: 0000<br>
Serial		: 0000000000000000<br>
</code></pre>
Работу можно еще упростить, если определить alias (псевдоним) для сокращенного набора. Например<br>
<pre><code>alias oem="ssh root@192.168.0.136"<br>
</code></pre>
теперь, чтобы выполнить команду на удаленной системе, в терминале хост-системы достаточно набрать<br>
<pre><code>oem cat /proc/cpuinfo <br>
Processor	: ARM926EJ-S rev 5 (v5l)<br>
BogoMIPS	: 199.06<br>
Features	: swp half thumb fastmult edsp java <br>
CPU implementer	: 0x41<br>
CPU architecture: 5TEJ<br>
CPU variant	: 0x0<br>
CPU part	: 0x926<br>
CPU revision	: 5<br>
<br>
Hardware	: Atmel AT91SAM9M10G45-EK<br>
Revision	: 0000<br>
Serial		: 0000000000000000<br>
</code></pre>
Если нужно, чтобы псевдоним сохранялся после перезагрузок хост-системы<br>
<pre><code>echo 'alias oem="ssh root@192.168.0.136"' &gt;&gt; ~/.bashrc<br>
</code></pre>
В дальнейшем вы увидите насколько удобно работать в консоли.