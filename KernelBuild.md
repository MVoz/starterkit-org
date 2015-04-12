# Введение #

Ядро может собираться как отдельно, так и в составе [buildroot](QtBuildroot.md). В некоторых случаях, отдельная сборка может оказаться удобнее. Описана сборка ядра 3.7.10 для платы SK-MAT91SAM9G45.

# Подготовка исходников ядра к сборке #

После скачивания исходников ядра с ftp.kernel.org их следует подготовить к работе. Особенность такова, что для не x86 требуется корректировка конфигурации в исходниках. Речь о так называемых board-файлах в каталоге arch. Для SK-MAT91SAM9G45 почти подходит arch/arm/mach-at91/board-sam9m10g45ek.c. Файл надо исправить по образцу того, что находится в прилагаемом к плате. Мне были нужны работающие SD, NAND, сетевая карта и USART (ttyS1-ttyS3). USB и RS-232, выведенный на разъём DB9 (ttyS0), работают без исправлений. Кому этого достаточно, могут воспользоваться готовым патчем [linux-3.7.10-SK-MAT91SAM9G45.patch](http://starterkit-org.googlecode.com/files/linux-3.7.10-SK-MAT91SAM9G45.patch). Патч копируется в корень дерева исходников ядра, после чего выполняется команда
```
patch -p1 < linux-3.7.10-SK-MAT91SAM9G45.patch
```
Кроме того, патч отключает вывод предупреждения "atmel\_nand: Fall back to CPU I/O".<br>
До или после корректировки board-файла следует выполнить начальное конфигурирование исходников:<br>
<pre><code>make ARCH=arm at91sam9g45_defconfig<br>
</code></pre>
После этого можно приступить к донастройке конфигурации ядра под свои нужды:<br>
<pre><code>make ARCH=arm menuconfig<br>
</code></pre>

Можно воспользоваться моим конфигом в качестве примера: <a href='http://starterkit-org.googlecode.com/files/config-3.7.10-SK-MAT91SAM9G45-example'>config-3.7.10-SK-MAT91SAM9G45-example</a>. Конфиг следует скопировать в корень дерева исходников и переименовать в ".config". Если конфиг будет применяться к более новому ядру, следует, после переименования, выполнить<br>
<pre><code>make ARCH=arm oldconfig<br>
</code></pre>
и сделать выбор для дополнительно появившихся опций, если они будут.<br>
<br>
<h1>Сборка ядра</h1>

Предполагается, что в переменной PATH уже присутствует путь к <a href='CrosstoolNg.md'>кросскомпилятору</a> и <b>mkimage</b> из <a href='UBoot.md'>U-Boot</a>. <font color='#CC0033'>Следует обратить внимание на то, что <b>mkimage</b> - удобное имя, и в системе может оказаться другое приложение с аналогичным названием.</font> В этом случае, можно скорректировать переменную PATH для пользователя, из-под которого производится сборка, или принять какие-то другие меры. Сборку можно выполнить следующей командой:<br>
<pre><code>make ARCH=arm CROSS_COMPILE=arm-atmel-linux-gnueabi- uImage<br>
</code></pre>
CROSS_COMPILE - это префикс тулчейна, который получился после сборки <a href='CrosstoolNg.md'>кросскомпилятора</a>. В случае успешной сборки ядра, в каталоге arch/arm/boot будет лежать готовое ядро uImage, пригодное к загрузке как с SD-карты, так и из NAND на плате, с нижеописанными нюансами.<br>
<br>
<h1>Настройка параметров загрузки ядра</h1>

При загрузке с разных носителей ядро должно использовать разные строки параметров.<br>
Пример для NAND:<br>
<pre><code>console=ttyS0,115200 ubi.mtd=1 root=ubi0:nandfs rw rootfstype=ubifs mem=64M<br>
</code></pre>
Пример для SD:<br>
<pre><code>console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait mem=64M debug<br>
</code></pre>
Строку параметров можно задать как в ядре, в момент конфигурирования, так и в загрузчиках <a href='Bootstrap.md'>bootstrap</a> и <a href='UBoot.md'>u-boot</a>:<br>
<pre><code>Boot options  ---&gt;<br>
  -*- Flattened Device Tree support<br>
  [*]   Support for the traditional ATAGS boot data passing<br>
  [ ]     Provide old way to pass kernel parameters<br>
  (0x0) Compressed ROM boot loader base address<br>
  (0x0) Compressed ROM boot loader BSS address<br>
  [ ] Use appended device tree blob to zImage (EXPERIMENTAL)<br>
  (console=ttyS0,115200 ubi.mtd=1 root=ubi0:nandfs rw rootfstype=ubifs) Default kernel command string<br>
        Kernel command line type (Use bootloader kernel arguments if available)  ---&gt;<br>
              (X) Use bootloader kernel arguments if available<br>
              ( ) Extend bootloader kernel arguments                  <br>
              ( ) Always use the default kernel command string<br>
  [ ] Kernel Execute-In-Place from ROM<br>
  [ ] Kexec system call (EXPERIMENTAL)<br>
  [ ] Build kdump crash kernel (EXPERIMENTAL)<br>
  [*] Auto calculation of the decompressed kernel image address<br>
</code></pre>
Выбор "Use bootloader kernel arguments if available", как раз, позволяет иметь одинаковое ядро, но при условии, что правильная строка передаётся из загрузчика. Если же хочется использовать строку "Default kernel command string", необходимо или выбрать "Always use the default kernel command string", или отключить передачу параметров загрузки из загрузчика.