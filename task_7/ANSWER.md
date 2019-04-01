# Ответ на вопрос №7

## Огромное спасибо Роману @Romanbullet 

Ответ Романа в файле **ANSWER.pdf**


## Небольшой оффтоп от~~для~~ себя

Как и у Романа, мой выбор пал на **Ubuntu**

* ```apt install volatility```
* Узнаем версию ОС ```volatility -f 20190227.mem imageinfo```:
```
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/root/20190227.mem)
                      PAE type : PAE
                           DTB : 0x334000L
                          KDBG : 0x8054c2e0L
          Number of Processors : 1
     Image Type (Service Pack) : 2
                KPCR for CPU 0 : 0xffdff000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2019-02-27 13:33:47 UTC+0000
     Image local date and time : 2019-02-27 16:33:47 +0300
```
* Смотрим, что лежит на рабочем столе с расширением **.doc**: ```volatility -f 20190227.mem --profile=WinXPSP2x86 windows | grep .doc```:
```$
Volatility Foundation Volatility Framework 2.6
Window Handle: #5010c at 0xbc676728, Name: Секретно.doc - Microsoft Word
Window Handle: #20160 at 0xbc6781c8, Name: Секретно.doc
```
* Делаем dump файла к себе на машину ```volatility -f 20190227.mem --profile=WinXPSP2x86 dumpfiles -r Секретно.doc --name -D ./ -u``` 

И открываем полученный файл **file.1420.0x82237ba8.Секретно.doc.dat** 