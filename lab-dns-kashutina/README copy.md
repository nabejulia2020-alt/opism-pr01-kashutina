# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Кашутіна Юлія Петрівна |
| Група | ЗКН-21 |
| Ідентифікатор (`sNN`) | `se-1-14` |
| Дата виконання | 21.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu |

> Команди виконувалися з термінала Codespaces у контейнері стенда через обгортку `r() { docker compose -f stand/compose.yaml exec -T se-1-14 bash -lic "$*"; }`. Перед кожним виводом наведено рядок `$ команда`. Привітальний банер стенда (перелік команд `zone-edit`, `zone-reload` тощо), який оболонка друкує перед кожною командою, з виводів вилучено, решту наведено без змін. Час у виводах — UTC.

## Параметри варіанта

| Параметр | Значення |
|---|---|
| `$TTL`, с | 90 |
| TTL запису SOA, с | 120 |
| Поле `minimum` SOA, с | 20 |
| Псевдонім (CNAME на `www`) | `app` |
| Другий хост і його TTL, с | `edge`, 25 |
| Пріоритет MX | 35 |
| Токен TXT | `lab-token=27ccff46` |
| Ім'я для завдання 5 | `new-ydmis.se-1-14.lab` |

## Завдання 1. Складові URI

### 1.1. Розбір

URI з картки:

1. `http://www.se-1-14.lab/search/reports?lang=71&sort=asc#top`
2. `http://app.se-1-14.lab:80/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2`
3. `http://user:pass@edge.se-1-14.lab/search#top`
4. `https://www.se-1-14.lab:4443/search/../reports`
5. `mailto:admin@se-1-14.lab?subject=search`

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | `http` | — | `www.se-1-14.lab` | 80 (за замовч.) | `/search/reports` | `lang=71&sort=asc` | `top` |
| 2 | `http` | — | `app.se-1-14.lab` | 80 | `/reports/` | `lang=%D0%BA%D0%B8%D1%97%D0%B2` | — |
| 3 | `http` | `user:pass` | `edge.se-1-14.lab` | 80 (за замовч.) | `/search` | — | `top` |
| 4 | `https` | — | `www.se-1-14.lab` | 4443 | `/search/../reports` | — | — |
| 5 | `mailto` | — | — | — | `admin@se-1-14.lab` | `subject=search` | — |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | `www.se-1-14.lab` | `www.se-1-14.lab` | `/search/reports?lang=71&sort=asc` |
| 2 | `app.se-1-14.lab` | `app.se-1-14.lab` (порт 80 стандартний, тому не додається) | `/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2` |
| 3 | `edge.se-1-14.lab` | `edge.se-1-14.lab` | `/search` |
| 4 | `www.se-1-14.lab` | `www.se-1-14.lab:4443` | `/reports` (клієнт нормалізує `/search/../reports`) |
| 5 | — (це не HTTP-запит; поштова програма шукала б MX для `se-1-14.lab`) | — | — |

Декодоване значення параметра URI 2: `lang=київ` (байти UTF-8 `D0 BA D0 B8 D1 97 D0 B2` — літери «к», «и», «ї», «в»).

Чому URI 5 не має складової host, хоча містить символ `@`: authority (і разом з ним host) є лише тоді, коли після `scheme:` стоїть `//`. У `mailto:admin@se-1-14.lab` подвійної косої немає, тому весь рядок `admin@se-1-14.lab` — це path, а `@` — звичайний символ адреси пошти.

### 1.2. Перевірка (після завдання 3)

```text
$ curl -v "http://www.se-1-14.lab/search/reports?lang=71&sort=asc#top"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host www.se-1-14.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-1-14.lab (10.53.0.80) port 80
> GET /search/reports?lang=71&sort=asc HTTP/1.1
> Host: www.se-1-14.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sat, 21 Nov 2026 17:18:42 GMT
< Content-Type: text/plain
< Content-Length: 146
< Connection: keep-alive
< 
{ [146 bytes data]

100   146  100   146    0     0  74795      0 --:--:-- --:--:-- --:--:--  142k
server: web-a
Host: www.se-1-14.lab
request URI: /search/reports?lang=71&sort=asc
path: /search/reports
query: lang=71&sort=asc
client: 10.53.1.1
* Connection #0 to host www.se-1-14.lab left intact
$ curl -v "http://app.se-1-14.lab:80/reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host app.se-1-14.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to app.se-1-14.lab (10.53.0.80) port 80
> GET /reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: app.se-1-14.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sat, 21 Nov 2026 17:18:43 GMT
< Content-Type: text/plain
< Content-Length: 160
< Connection: keep-alive
< 
{ [160 bytes data]

100   160  100   160    0     0  38740      0 --:--:-- --:--:-- --server: web-a
Host: app.se-1-14.lab
request URI: /reports/?lang=%D0%BA%D0%B8%D1%97%D0%B2
path: /reports/
query: lang=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
:--:-- 53333
* Connection #0 to host app.se-1-14.lab left intact
$ curl -v "http://user:pass@edge.se-1-14.lab/search#top"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host edge.se-1-14.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to edge.se-1-14.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /search HTTP/1.1
> Host: edge.se-1-14.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sat, 21 Nov 2026 17:18:43 GMT
< Content-Type: text/plain
< Content-Length: 98
< Connection: keep-alive
< 
{ [98 bytes data]

100    98  100    98    0     0  45623      0 --:--:-- --:--:-- --:--:-- 49000
* Connection #0 to host edge.se-1-14.lab left intact
server: web-b
Host: edge.se-1-14.lab
request URI: /search
path: /search
query: 
client: 10.53.1.1
```

Значення заголовка `Authorization` і результат декодування:

```text
> Authorization: Basic dXNlcjpwYXNz
$ echo 'dXNlcjpwYXNz' | base64 -d
user:pass
```

URI 4: запит не вдається на етапі **TCP**. Ім'я успішно перетворено на адресу (`was resolved`, `IPv4: 10.53.0.80`), але з'єднання з портом 4443 відхилено (`Connection refused`), тож до TLS справа не дійшла:

```text
$ curl -v "https://www.se-1-14.lab:4443/search/../reports"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host www.se-1-14.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:4443...
* connect to 10.53.0.80 port 4443 from 10.53.1.1 port 46288 failed: Connection refused
* Failed to connect to www.se-1-14.lab port 4443 after 1 ms: Couldn't connect to server

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection
curl: (7) Failed to connect to www.se-1-14.lab port 4443 after 1 ms: Couldn't connect to server
```

Перевірка шляху з `..`:

```text
$ curl -v "http://www.se-1-14.lab/search/../reports"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host www.se-1-14.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-1-14.lab (10.53.0.80) port 80
> GET /reports HTTP/1.1
> Host: www.se-1-14.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sat, 21 Nov 2026 17:19:02 GMT
< Content-Type: text/plain
< Content-Length: 99
< Connection: keep-alive
< 
{ [99 bytes data]

100    99  100    99    0     0  75745      0 --:--:-- --:--:-- --:--:-- 99000
* Connection #0 to host www.se-1-14.lab left intact
server: web-a
Host: www.se-1-14.lab
request URI: /reports
path: /reports
query: 
client: 10.53.1.1
$ curl -v --path-as-is "http://www.se-1-14.lab/search/../reports"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host www.se-1-14.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-1-14.lab (10.53.0.80) port 80
> GET /search/../reports HTTP/1.1
> Host: www.se-1-14.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sat, 21 Nov 2026 17:19:02 GMT
< Content-Type: text/plain
< Content-Length: 109
< Connection: keep-alive
< 
{ [109 bytes data]

100   109  100   109    0     0  69338      0 --:--:-- --:--:-- --:--:--  106k
* Connection #0 to host www.se-1-14.lab left intact
server: web-a
Host: www.se-1-14.lab
request URI: /search/../reports
path: /reports
query: 
client: 10.53.1.1
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | `/reports` | `/reports` |
| `--path-as-is` | `/search/../reports` | `/reports` |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:

- Fragment `#top` (URI 1 і 3) до сервера не дійшов: у `request URI` його немає, бо клієнт ніколи не надсилає fragment.
- `user:pass` з URI 3 не потрапив у рядок запиту, а перетворився на заголовок `Authorization: Basic dXNlcjpwYXNz`.
- У URI 2 явний порт `:80` у заголовку `Host` не з'явився, бо він стандартний для `http`.
- Шлях `/search/../reports` curl сам скоротив до `/reports`; лише з `--path-as-is` сервер отримав його як є (і nginx уже сам нормалізував `path` до `/reports`).
- Кирилиця в URI 2 передана у відсотковому кодуванні без змін.

## Завдання 2. Проходження ієрархії вручну

```text
$ dig @10.53.0.2 ns.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.2 ns.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22112
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8068a6e24462738f010000006b01cfbddb608d6b4393c99a (good)
;; QUESTION SECTION:
;ns.se-1-14.lab.			IN	A

;; AUTHORITY SECTION:
lab.			172800	IN	NS	ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.		172800	IN	A	10.53.0.3

;; Query time: 80 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Sat Nov 21 17:07:09 UTC 2026
;; MSG SIZE  rcvd: 115

$ dig @10.53.0.3 ns.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.3 ns.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9693
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b408af7200e89afb010000006b01cfbe05db633bac132327 (good)
;; QUESTION SECTION:
;ns.se-1-14.lab.			IN	A

;; AUTHORITY SECTION:
se-1-14.lab.		3600	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		3600	IN	A	10.53.1.1

;; Query time: 80 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sat Nov 21 17:07:10 UTC 2026
;; MSG SIZE  rcvd: 101

$ dig @10.53.1.1 ns.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 ns.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13175
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: eaba6d70178a63b8010000006b01cfbec504f8edbf505a7f (good)
;; QUESTION SECTION:
;ns.se-1-14.lab.			IN	A

;; ANSWER SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:07:10 UTC 2026
;; MSG SIZE  rcvd: 101
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | `qr` | — | `lab. 172800 NS ns1.nic.lab.` | `ns1.nic.lab. 172800 A 10.53.0.3` |
| 10.53.0.3 | NOERROR | `qr` | — | `se-1-14.lab. 3600 NS ns.se-1-14.lab.` | `ns.se-1-14.lab. 3600 A 10.53.1.1` |
| 10.53.1.1 | NOERROR | `qr aa` | `ns.se-1-14.lab. 90 A 10.53.1.1` | `se-1-14.lab. 90 NS ns.se-1-14.lab.` | — |

(Лічильник ADDITIONAL у заголовку dig більший на 1, бо враховує псевдозапис OPT.)

Вивід `dig ns.se-1-14.lab A +trace`:

```text
$ dig ns.se-1-14.lab A +trace


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> ns.se-1-14.lab A +trace
;; global options: +cmd
.			86138	IN	NS	a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.			172800	IN	NS	ns1.nic.lab.
;; Received 115 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-1-14.lab.		3600	IN	NS	ns.se-1-14.lab.
;; Received 101 bytes from 10.53.0.3#53(ns1.nic.lab) in 39 ms

ns.se-1-14.lab.		90	IN	A	10.53.1.1
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.
;; Received 101 bytes from 10.53.1.1#53(ns.se-1-14.lab) in 0 ms
```

```text
$ dig @10.53.0.3 se-1-14.lab NS +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.3 se-1-14.lab NS +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42240
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f5c085dbb6a68490010000006b01cfd6b2109d461ac5266c (good)
;; QUESTION SECTION:
;se-1-14.lab.			IN	NS

;; AUTHORITY SECTION:
se-1-14.lab.		3600	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		3600	IN	A	10.53.1.1

;; Query time: 41 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sat Nov 21 17:07:34 UTC 2026
;; MSG SIZE  rcvd: 101

$ dig @10.53.1.1 se-1-14.lab NS +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-1-14.lab NS +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2662
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 5a085ba67ebb7b52010000006b01cfd6cca7548dece78697 (good)
;; QUESTION SECTION:
;se-1-14.lab.			IN	NS

;; ANSWER SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:07:34 UTC 2026
;; MSG SIZE  rcvd: 101
```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 se-1-14.lab NS +norec` | 3600 | немає | AUTHORITY |
| `dig @10.53.1.1 se-1-14.lab NS +norec` | 90 | є | ANSWER |

1. Які відповіді є відсиланнями і за якими ознаками це визначено?

   Відповіді root (10.53.0.2) і TLD (10.53.0.3). Ознаки: секція ANSWER порожня (`ANSWER: 0`), в AUTHORITY є записи NS наступного рівня (`lab.` → `ns1.nic.lab.`, `se-1-14.lab.` → `ns.se-1-14.lab.`), а прапорця `aa` немає (`flags: qr`). Тобто сервер не відповідає сам, а каже, кого питати далі.

2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?

   Це glue-запис `ns.se-1-14.lab. A 10.53.1.1`. Сервер моєї зони має ім'я всередині самої зони, тому без glue резолвер, щоб дізнатися адресу `ns.se-1-14.lab`, мусив би спитати `ns.se-1-14.lab` — замкнене коло, і ім'я не зарезолвилося б.

3. Яка з двох копій запису NS є авторитативною і чому?

   Копія від 10.53.1.1 (TTL 90, прапорець `aa`, секція ANSWER). Саме мій сервер відповідає за зону `se-1-14.lab`. Копія в TLD (TTL 3600, без `aa`, у AUTHORITY) — лише делегування в батьківській зоні, підказка, куди йти.

## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```text
$ zone-reload

zone se-1-14.lab/IN: loaded serial 2026112102
OK
zone reload queued
```

```text
$ dig @10.53.1.1 www.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 www.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24520
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ca9d20d8ad0ae82a010000006b01d1c51829435a0a3fd9f4 (good)
;; QUESTION SECTION:
;www.se-1-14.lab.		IN	A

;; ANSWER SECTION:
www.se-1-14.lab.	90	IN	A	10.53.0.80

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:49 UTC 2026
;; MSG SIZE  rcvd: 121

$ dig @10.53.1.1 app.se-1-14.lab CNAME +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 app.se-1-14.lab CNAME +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39914
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b7e55a62f8025803010000006b01d1c521e546a5891385c9 (good)
;; QUESTION SECTION:
;app.se-1-14.lab.		IN	CNAME

;; ANSWER SECTION:
app.se-1-14.lab.	90	IN	CNAME	www.se-1-14.lab.

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:49 UTC 2026
;; MSG SIZE  rcvd: 123

$ dig @10.53.1.1 mail.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 mail.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6259
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: de483971e71dc49d010000006b01d1c5e2ce752efdc6b503 (good)
;; QUESTION SECTION:
;mail.se-1-14.lab.		IN	A

;; ANSWER SECTION:
mail.se-1-14.lab.	90	IN	A	10.53.0.80

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:49 UTC 2026
;; MSG SIZE  rcvd: 122

$ dig @10.53.1.1 se-1-14.lab MX +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-1-14.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25455
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7648c9e583721fe4010000006b01d1c5f808a22c6edae4e8 (good)
;; QUESTION SECTION:
;se-1-14.lab.			IN	MX

;; ANSWER SECTION:
se-1-14.lab.		90	IN	MX	35 mail.se-1-14.lab.

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
mail.se-1-14.lab.	90	IN	A	10.53.0.80
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:49 UTC 2026
;; MSG SIZE  rcvd: 138

$ dig @10.53.1.1 se-1-14.lab TXT +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-1-14.lab TXT +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27444
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3c201797bd070928010000006b01d1c526aecdf0b9671602 (good)
;; QUESTION SECTION:
;se-1-14.lab.			IN	TXT

;; ANSWER SECTION:
se-1-14.lab.		90	IN	TXT	"lab-token=27ccff46"

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:49 UTC 2026
;; MSG SIZE  rcvd: 132

$ dig @10.53.1.1 edge.se-1-14.lab A +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 edge.se-1-14.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36410
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0e653351a7d265a2010000006b01d1c647a93647e49674be (good)
;; QUESTION SECTION:
;edge.se-1-14.lab.		IN	A

;; ANSWER SECTION:
edge.se-1-14.lab.	25	IN	A	10.53.0.81

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:50 UTC 2026
;; MSG SIZE  rcvd: 122
```

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | так | `10.53.0.80` | 90 | так |
| псевдонім CNAME | NOERROR | так | `app` → `www.se-1-14.lab.` | 90 | так |
| `mail` A | NOERROR | так | `10.53.0.80` | 90 | так |
| `@` MX | NOERROR | так | `35 mail.se-1-14.lab.` | 90 | так |
| `@` TXT | NOERROR | так | `"lab-token=27ccff46"` | 90 | так |
| другий хост A | NOERROR | так | `edge` → `10.53.0.81` | 25 | так |

Порівняння відповідей резолвера та власного сервера:

```text
$ dig app.se-1-14.lab


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> app.se-1-14.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4204
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;app.se-1-14.lab.		IN	A

;; ANSWER SECTION:
app.se-1-14.lab.	90	IN	CNAME	www.se-1-14.lab.
www.se-1-14.lab.	90	IN	A	10.53.0.80

;; Query time: 40 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sat Nov 21 17:15:58 UTC 2026
;; MSG SIZE  rcvd: 78

$ dig @10.53.1.1 app.se-1-14.lab +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 app.se-1-14.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61128
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 398b9a5a7c62fa24010000006b01d1ce8b88d1c5d1d1f415 (good)
;; QUESTION SECTION:
;app.se-1-14.lab.		IN	A

;; ANSWER SECTION:
app.se-1-14.lab.	90	IN	CNAME	www.se-1-14.lab.
www.se-1-14.lab.	90	IN	A	10.53.0.80

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:58 UTC 2026
;; MSG SIZE  rcvd: 139

$ dig se-1-14.lab MX


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> se-1-14.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1720
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;se-1-14.lab.			IN	MX

;; ANSWER SECTION:
se-1-14.lab.		90	IN	MX	35 mail.se-1-14.lab.

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sat Nov 21 17:15:58 UTC 2026
;; MSG SIZE  rcvd: 61

$ dig @10.53.1.1 se-1-14.lab MX +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-1-14.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38160
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a676d221a60618e1010000006b01d1cefb8d9349ab9433f9 (good)
;; QUESTION SECTION:
;se-1-14.lab.			IN	MX

;; ANSWER SECTION:
se-1-14.lab.		90	IN	MX	35 mail.se-1-14.lab.

;; AUTHORITY SECTION:
se-1-14.lab.		90	IN	NS	ns.se-1-14.lab.

;; ADDITIONAL SECTION:
mail.se-1-14.lab.	90	IN	A	10.53.0.80
ns.se-1-14.lab.		90	IN	A	10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:15:58 UTC 2026
;; MSG SIZE  rcvd: 138
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | `qr rd ra` | 2: CNAME, A | 0 | 0 (лише OPT) | 90 |
| псевдонім через 10.53.1.1 | `qr aa` | 2: CNAME, A | 1: NS | 1: A (`ns`) | 90 |
| MX через resolver-a | `qr rd ra` | 1: MX | 0 | 0 (лише OPT) | 90 |
| MX через 10.53.1.1 | `qr aa` | 1: MX | 1: NS | 2: A (`mail`), A (`ns`) | 90 |

Пояснення відмінностей: резолвер ставить `ra` (рекурсія доступна) і не ставить `aa`, бо не є власником зони, а віддає відповідь, отриману від мого сервера. Авторитативний сервер додає в AUTHORITY запис NS своєї зони, а в ADDITIONAL — адреси згаданих імен (`ns`, `mail`), щоб клієнту не довелося питати їх окремо; резолвер ці «довідкові» секції клієнту не передає. TTL однаковий (90), бо резолвер щойно отримав записи і ще не встиг їх «зістарити».

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
$ zone-check

dns_master_load: /lab/zone/se-1-14.lab.zone:21: se-1-14.lab: CNAME and other data
zone se-1-14.lab/IN: loading from master file /lab/zone/se-1-14.lab.zone failed: CNAME and other data
zone se-1-14.lab/IN: not loaded due to errors.
```

1. Який TTL має кожен запис зони і звідки це значення взято?

   Усі записи, крім `edge`, мають TTL 90 — це значення `$TTL 90` на початку файлу зони, бо власного TTL у рядках немає. `edge` має TTL 25, бо він явно вказаний у рядку `edge 25 IN A`. Сам SOA має TTL 120, вказаний у його рядку.

2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?

   Два (`ANSWER: 2`): `app CNAME www.se-1-14.lab.` і `www A 10.53.0.80`. Запитували тип A; для `app` є лише CNAME, тож сервер повертає його і одразу дописує запис A для імені, на яке вказує псевдонім, бо ця зона в нього є.

3. Чому запис CNAME неможливо розмістити для імені `@`?

   На імені `@` (корені зони) обов'язково є SOA і NS, а CNAME не може співіснувати з іншими записами того самого імені. Саме це повідомляє `zone-check`: `CNAME and other data`, і зона не завантажується.

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
$ dig www.se-1-14.lab +noall +answer

www.se-1-14.lab.	90	IN	A	10.53.0.80
$ dig www.se-1-14.lab +noall +answer

www.se-1-14.lab.	85	IN	A	10.53.0.80
```

**Прогноз** (записано до вимірювання). Нова адреса з'явиться через резолвер у момент t(зміни) + R, де R — залишок TTL запису в кеші резолвера на момент зміни. Найбільша можлива затримка дорівнює повному TTL запису, 90 с.

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
$ dig @10.53.1.1 www.se-1-14.lab +norec +short

10.53.0.80
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text
$ date +%T; dig www.se-1-14.lab +noall +answer

17:24:14
www.se-1-14.lab.	90	IN	A	10.53.0.80
$ zone-reload; date +%T

zone se-1-14.lab/IN: loaded serial 2026112103
OK
zone reload queued
17:24:14
$ dig @10.53.1.1 www.se-1-14.lab +norec +short

10.53.0.81
```

Очікуваний час появи нової адреси: R = 90 с (запис щойно потрапив у кеш), отже 17:24:14 + 90 с = **17:25:44**.

Вивід циклу спостереження (повністю):

```text
$ while true; do a=$(dig +short www.se-1-14.lab); echo "$(date +%T) $a"; [ "$a" = "10.53.0.81" ] && break; sleep 2; done

17:24:14 10.53.0.80
17:24:16 10.53.0.80
17:24:18 10.53.0.80
17:24:21 10.53.0.80
17:24:23 10.53.0.80
17:24:25 10.53.0.80
17:24:27 10.53.0.80
17:24:29 10.53.0.80
17:24:31 10.53.0.80
17:24:33 10.53.0.80
17:24:35 10.53.0.80
17:24:37 10.53.0.80
17:24:39 10.53.0.80
17:24:41 10.53.0.80
17:24:43 10.53.0.80
17:24:45 10.53.0.80
17:24:47 10.53.0.80
17:24:49 10.53.0.80
17:24:51 10.53.0.80
17:24:53 10.53.0.80
17:24:55 10.53.0.80
17:24:57 10.53.0.80
17:24:59 10.53.0.80
17:25:01 10.53.0.80
17:25:03 10.53.0.80
17:25:05 10.53.0.80
17:25:07 10.53.0.80
17:25:09 10.53.0.80
17:25:11 10.53.0.80
17:25:13 10.53.0.80
17:25:15 10.53.0.80
17:25:17 10.53.0.80
17:25:19 10.53.0.80
17:25:21 10.53.0.80
17:25:23 10.53.0.80
17:25:25 10.53.0.80
17:25:27 10.53.0.80
17:25:29 10.53.0.80
17:25:31 10.53.0.80
17:25:33 10.53.0.80
17:25:35 10.53.0.80
17:25:37 10.53.0.80
17:25:39 10.53.0.80
17:25:41 10.53.0.80
17:25:43 10.53.0.80
17:25:45 10.53.0.81
```

Рядок `server:` з відповіді `curl`: `server: web-b`

```text
$ curl -s http://www.se-1-14.lab/

server: web-b
Host: www.se-1-14.lab
request URI: /
path: /
query: 
client: 10.53.1.1
```

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 17:24:14 | 17:25:44 | 17:25:45 | 91 | 90 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

   Розбіжність 1 с. Цикл опитує резолвер кожні ~2 с (видно 17:25:43 і 17:25:45), тож момент появи визначається з точністю до кроку опитування; ще додається час виконання самих команд і те, що `date` показує цілі секунди.

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

   90 с — повний TTL. Так буде, якщо резолвер закешував стару адресу безпосередньо перед зміною (як і сталося тут: запит о 17:24:14 отримав TTL 90). Якщо ланцюжок кешів довший (кеш ОС, браузера, пул з'єднань застосунку), реальна затримка може бути ще більшою.

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

   Заздалегідь, щонайменше за 86 400 с (добу) до переїзду, знизити TTL запису до кількох десятків секунд; дочекатися, поки старі копії з довгим TTL спливуть; змінити адресу (старий сервер на час переходу не вимикати); перевірити, що клієнти перейшли; повернути TTL до звичного значення.

## Завдання 5. Негативне кешування

**Прогноз** (записано до вимірювання). За RFC 2308 негативна відповідь кешується на min(TTL запису SOA, поле `minimum` SOA) = min(120, 20) = **20 с**.

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
$ dig @10.53.1.1 new-ydmis.se-1-14.lab +norec


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 new-ydmis.se-1-14.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 13528
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b922e14b27efa95e010000006b01d4795b860f1e27c6e8de (good)
;; QUESTION SECTION:
;new-ydmis.se-1-14.lab.		IN	A

;; AUTHORITY SECTION:
se-1-14.lab.		20	IN	SOA	ns.se-1-14.lab. admin.se-1-14.lab. 2026112103 3600 600 86400 20

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sat Nov 21 17:27:21 UTC 2026
;; MSG SIZE  rcvd: 123
```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text
$ date +%T; dig new-ydmis.se-1-14.lab

17:27:22

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> new-ydmis.se-1-14.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 36648
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-ydmis.se-1-14.lab.		IN	A

;; AUTHORITY SECTION:
se-1-14.lab.		20	IN	SOA	ns.se-1-14.lab. admin.se-1-14.lab. 2026112103 3600 600 86400 20

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sat Nov 21 17:27:22 UTC 2026
;; MSG SIZE  rcvd: 95

$ zone-reload; date +%T

zone se-1-14.lab/IN: loaded serial 2026112104
OK
zone reload queued
17:27:22
$ dig @10.53.1.1 new-ydmis.se-1-14.lab +norec +short

10.53.0.80
```

Вивід циклу спостереження (повністю):

```text
$ while true; do s=$(dig new-ydmis.se-1-14.lab +noall +comments | grep -o "status: [A-Z]*"); echo "$(date +%T) $s"; [ "$s" = "status: NOERROR" ] && break; sleep 2; done

17:27:22 status: NXDOMAIN
17:27:24 status: NXDOMAIN
17:27:26 status: NXDOMAIN
17:27:28 status: NXDOMAIN
17:27:30 status: NXDOMAIN
17:27:32 status: NXDOMAIN
17:27:34 status: NXDOMAIN
17:27:36 status: NXDOMAIN
17:27:38 status: NXDOMAIN
17:27:40 status: NXDOMAIN
17:27:42 status: NXDOMAIN
17:27:44 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 17:27:22 | 17:27:22 | 17:27:44 | 22 | 20 |

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

   Запиту `new-ydmis.se-1-14.lab` не існувало, тому й власного TTL у нього немає. Відповідь «імені немає» описує стан зони, і сервер додає до неї SOA зони (видно в AUTHORITY), щоб резолвер знав, скільки цю відповідь можна зберігати.

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

   Ні. У файлі SOA має TTL 120, а в AUTHORITY — 20. Сервер для негативної відповіді ставить min(TTL SOA, `minimum`) = min(120, 20) = 20. Через резолвер це значення в кеші ще й зменшувалося б щосекунди.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

   Після створення запису скрипт ще до 20 с (тут виміряно 22 с) отримуватиме `NXDOMAIN` з кешу резолвера, хоча запис уже є на сервері, і вирішить, що розгортання не вдалося. Виправлення: не запитувати ім'я до його створення; перевіряти результат запитом безпосередньо до авторитативного сервера з `+norec`; або чекати довше за негативний TTL і повторювати перевірку.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записано до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | лише власний сервер (делегування `se-1-14.lab` уже в кеші) | жодного (з кешу, негативна відповідь) |
| B: нове ім'я в зоні `lab.` | лише TLD | жодного (з кешу) |
| C: ім'я в неіснуючому TLD | лише root | жодного (з кешу) |
| D: `www.se-1-14.lab` після перезапуску резолвера | root → TLD → власний сервер | жодного (з кешу) |

Вивід вимірювань:

```text
$ A=y$RANDOM.se-1-14.lab; echo $A; dig $A | grep "Query time"; dig $A | grep "Query time"

y5548.se-1-14.lab
;; Query time: 1 msec
;; Query time: 0 msec
$ B=x$RANDOM.lab; echo $B; dig $B | grep "Query time"; dig $B | grep "Query time"

x22583.lab
;; Query time: 41 msec
;; Query time: 0 msec
$ C=x$RANDOM.zz$RANDOM; echo $C; dig $C | grep "Query time"; dig $C | grep "Query time"

x20704.zz22489
;; Query time: 80 msec
;; Query time: 0 msec

$ docker compose -f stand/compose.yaml restart resolver-a; sleep 5
$ dig www.se-1-14.lab | grep "Query time"; dig www.se-1-14.lab | grep "Query time"

;; Query time: 402 msec
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A (`y5548.se-1-14.lab`) | 1 | 0 | ні root, ні TLD (у журналах root і TLD цього імені немає); лише власний сервер |
| B (`x22583.lab`) | 41 | 0 | TLD (`17:29:00.161 … x22583.lab IN A … (10.53.0.3)`) |
| C (`x20704.zz22489`) | 80 | 0 | root (`17:29:00.390 … x20704.zz22489 IN A … (10.53.0.2)`) |
| D (`www.se-1-14.lab`) | 402 | 0 | root (`. NS`, `a.root-servers.net A`, `www.se-1-14.lab A`), TLD (`www.se-1-14.lab A`), власний сервер |

Значення `time_namelookup` з `curl` після перезапуску резолвера: **0.402767 с**

```text
$ curl -so /dev/null -w "%{time_namelookup}\n" http://www.se-1-14.lab/

0.402767
```

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
$ root: tail queries.log
21-Nov-2026 17:02:57.012 client @0x7251f4000e98 10.53.0.53#41563 (.): query: . IN NS -E(0)DC (10.53.0.2)
21-Nov-2026 17:02:57.091 client @0x7251f4000e98 10.53.0.53#44986 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:02:57.172 client @0x7251f806ac88 10.53.0.53#8188 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:02:57.596 client @0x7251f806ac88 10.53.0.54#48839 (.): query: . IN NS -E(0)DC (10.53.0.2)
21-Nov-2026 17:02:57.676 client @0x7251f806ac88 10.53.0.54#14749 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:02:57.757 client @0x7251f806ac88 10.53.0.54#54126 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:07:09.871 client @0x7251f4000e98 10.53.1.1#48675 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)K (10.53.0.2)
21-Nov-2026 17:07:19.849 client @0x7251f806ac88 10.53.1.1#35777 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DK (10.53.0.2)
21-Nov-2026 17:29:00.390 client @0x7251f806ac88 10.53.0.53#61637 (x20704.zz22489): query: x20704.zz22489 IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:29:06.248 client @0x7251f4000e98 10.53.0.53#51760 (.): query: . IN NS -E(0)DC (10.53.0.2)
21-Nov-2026 17:29:06.328 client @0x7251f4000e98 10.53.0.53#15307 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
21-Nov-2026 17:29:06.408 client @0x7251f4000e98 10.53.0.53#35844 (www.se-1-14.lab): query: www.se-1-14.lab IN A -E(0)DC (10.53.0.2)
$ tld: tail queries.log
21-Nov-2026 17:02:57.292 client @0x735e4006ac88 10.53.0.53#11526 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:02:57.878 client @0x735e4006ac88 10.53.0.54#50706 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:07:10.195 client @0x735e4006ac88 10.53.1.1#59042 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)K (10.53.0.3)
21-Nov-2026 17:07:19.930 client @0x735e4006ac88 10.53.0.53#52405 (ns1.nic.lab): query: ns1.nic.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:07:19.971 client @0x735e4006ac88 10.53.1.1#56195 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DK (10.53.0.3)
21-Nov-2026 17:07:20.011 client @0x735e4006ac88 10.53.0.53#37951 (ns.se-1-14.lab): query: ns.se-1-14.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:07:34.045 client @0x735e4006ac88 10.53.1.1#44721 (se-1-14.lab): query: se-1-14.lab IN NS -E(0)K (10.53.0.3)
21-Nov-2026 17:15:58.230 client @0x735e4006ac88 10.53.0.53#5077 (app.se-1-14.lab): query: app.se-1-14.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:29:00.161 client @0x735e4006ac88 10.53.0.53#50154 (x22583.lab): query: x22583.lab IN A -E(0)DC (10.53.0.3)
21-Nov-2026 17:29:06.529 client @0x735e3c0051f8 10.53.0.53#56112 (www.se-1-14.lab): query: www.se-1-14.lab IN A -E(0)DC (10.53.0.3)
$ timeout 3 querylog

timeout: failed to run command 'querylog': No such file or directory
```

Журнал власного сервера отримати не вдалося: команда `querylog` є функцією інтерактивної оболонки стенда і через `timeout` у неінтерактивному режимі не запустилася (`No such file or directory`). Те, що запити доходили до власного сервера, видно з відповідей `aa` і з того, що в журналах root/TLD немає запитів імені A.

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

   Root — близько 80 мс (рядок C: резолвер звертався лише до root, 80 мс; у `+trace` відповідь root теж 80 ms). TLD — близько 40 мс (рядок B: лише TLD, 41 мс; у `+trace` 39 ms). Власний сервер — близько 0–1 мс (рядок A: 1 мс). Рядок D (402 мс) більший за суму 80 + 40 + 1, бо після перезапуску в резолвера порожній кеш: за журналом root він спершу виконав priming — `. IN NS` і `a.root-servers.net IN A` (ще 2 × 80 мс), а також з'ясовував адреси серверів імен (`ns1.nic.lab`, `ns.se-1-14.lab`).

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

   Нове лише саме ім'я, а делегування зони `se-1-14.lab` (NS і glue з адресою 10.53.1.1) резолвер уже мав у кеші з попередніх завдань. Тому він одразу спитав мій сервер.

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

   Кожен новий домен — окремий промах кешу, який може коштувати від десятків до сотень мілісекунд (у моєму вимірі до 402 мс з холодним кешем), і лише після цього можна відкривати TCP/TLS-з'єднання. Частина пошуків іде паралельно, але перше відкриття сторінки все одно помітно сповільнюється; повторні відкриття беруть адреси з кешу (0 мс).

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записано до вимірювання). Після зупинки сервера і спливання TTL (25 с) resolver-a не зможе оновити запис і поверне SERVFAIL після кількох секунд спроб. resolver-b (serve-stale) швидко поверне старий запис `edge 10.53.0.81` з невеликим TTL (~30 с).

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text
$ dig @10.53.0.53 edge.se-1-14.lab +noall +answer

edge.se-1-14.lab.	25	IN	A	10.53.0.81
$ dig @10.53.0.54 edge.se-1-14.lab +noall +answer

edge.se-1-14.lab.	25	IN	A	10.53.0.81
$ named-stop; date +%T

17:30:14
```

Після зупинки зачекано 35 с (`sleep 35`), що більше за TTL запису `edge` (25 с).

Запити із заміром часу (крок 5):

```text
$ time dig @10.53.0.53 edge.se-1-14.lab +tries=1 +timeout=30


real	0m30.050s
user	0m0.004s
sys	0m0.006s
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 edge.se-1-14.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached
$ time dig @10.53.0.53 edge.se-1-14.lab +tries=1 +timeout=30

;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 edge.se-1-14.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real	0m30.051s
user	0m0.007s
sys	0m0.004s
$ time dig @10.53.0.54 edge.se-1-14.lab +tries=1 +timeout=30


; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.54 edge.se-1-14.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29661
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;edge.se-1-14.lab.		IN	A

;; ANSWER SECTION:
edge.se-1-14.lab.	30	IN	A	10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Sat Nov 21 17:31:52 UTC 2026
;; MSG SIZE  rcvd: 61


real	0m1.827s
user	0m0.006s
sys	0m0.009s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | тайм-аут (`communications error … timed out`, `no servers could be reached`) | — | 0m30.050s |
| resolver-a, другий | тайм-аут | — | 0m30.051s |
| resolver-b | NOERROR | `edge.se-1-14.lab. 30 IN A 10.53.0.81` | 0m1.827s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
$ named-start; date +%T

17:31:52
$ while true; do a=$(dig @10.53.0.53 edge.se-1-14.lab +short +tries=1 +timeout=3); echo "$(date +%T) $a"; [ "$a" = "10.53.0.81" ] && break; sleep 2; done

17:31:52 
17:31:54 
17:31:56 
17:31:58 
17:32:00 
17:32:02 
17:32:04 
17:32:06 
17:32:08 
17:32:10 
17:32:12 
17:32:15 
17:32:17 
17:32:19 
17:32:21 
17:32:23 10.53.0.81
```

Перевірка після завдання (`dig @10.53.1.1 se-1-14.lab SOA +norec +short`):

```text
$ dig @10.53.1.1 se-1-14.lab SOA +norec +short

ns.se-1-14.lab. admin.se-1-14.lab. 2026112104 3600 600 86400 20
```

### Постмортем

1. Що сталося: о 17:30:14 зупинено авторитативний сервер зони `se-1-14.lab` (10.53.1.1); зона стала недоступна для оновлення кешів.
2. Вплив на клієнтів resolver-a і resolver-b: клієнти resolver-a після спливання TTL взагалі не отримували відповіді — кожен запит чекав 30 с і завершувався тайм-аутом. Клієнти resolver-b отримали стару, але робочу адресу `10.53.0.81` приблизно за 1,8 с.
3. Хронологія з виміряним часом: 17:30:14 — сервер зупинено; ~17:30:39 — сплив TTL запису `edge` (25 с); 17:30:49–17:31:50 — два запити до resolver-a, кожен 30 с тайм-аут; 17:31:52 — resolver-b віддав застарілий запис з TTL 30; 17:31:52 — сервер запущено (`named-start`); 17:32:23 — resolver-a знову повернув `10.53.0.81` (через 31 с після запуску).
4. Чому кеш допоміг або не допоміг: для resolver-a кеш не допоміг, бо TTL запису короткий (25 с) і після його спливання резолвер не віддає прострочених даних. resolver-b допоміг механізм serve-stale: він віддав прострочений запис, коли не зміг дістатися до сервера.
5. Що слід змінити: мати щонайменше два авторитативні сервери зони в різних мережах; не ставити надто короткий TTL записам, без яких сервіс не працює; увімкнути serve-stale на резолверах; моніторити доступність DNS ззовні, з боку клієнта.

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

   30 с (хоча TTL запису в зоні 25 с і він уже сплив). Резолвер з serve-stale видає прострочений запис з коротким «stale TTL» (RFC 8767 рекомендує 30 с), щоб клієнти скоро перепитали і отримали свіжі дані, щойно сервер відновиться, але не закешували застарілу адресу надовго.

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

   Обидва запити однакові: жодної відповіді, тайм-аут клієнта через 30,05 с. Протягом цього часу резолвер намагається дістатися до авторитативного сервера 10.53.1.1: надсилає повторні запити з наростаючими інтервалами і чекає. Його власний ліміт на спроби, мабуть, довший за 30 с, тож він не встигає сформувати навіть SERVFAIL до того, як dig здається.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

   Після невдалих спроб резолвер позначив сервер 10.53.1.1 як недоступний (і, ймовірно, закешував невдачу) і деякий час не надсилає йому нових запитів, а ще продовжує обробку старих запитів, що «зависли». Лише через ~31 с він знову звернувся до сервера і отримав відповідь.

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

   Коли адреса справді змінилася, а авторитативний сервер у цей момент недоступний. Як у завданні 4: `www` переведено з 10.53.0.80 на 10.53.0.81. Якби після цього сервер зупинився, резолвер із serve-stale продовжував би роздавати стару адресу 10.53.0.80 — клієнти йшли б на сервер, який, можливо, вже вимкнено.

## Фінальний файл зони

```text
; Zone se-1-14.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 90
@     120 IN SOA ns.se-1-14.lab. admin.se-1-14.lab. (
            2026112104 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            20         ; minimum
        )
@     IN NS ns.se-1-14.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www       IN A      10.53.0.81
app       IN CNAME  www
mail      IN A      10.53.0.80
@         IN MX     35 mail
@         IN TXT    "lab-token=27ccff46"
edge  25  IN A      10.53.0.81
new-ydmis IN A      10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Я очікувала, що після спливання TTL resolver-a поверне SERVFAIL за кілька секунд, але він не відповів узагалі: обидва запити завершилися тайм-аутом клієнта (`real 0m30.050s`, `;; no servers could be reached`), тобто клієнт без serve-stale чекав пів хвилини і не отримав навіть повідомлення про помилку. Також несподіваною була затримка рядка D (`Query time: 402 msec`, `time_namelookup` 0.402767) — у кілька разів більша за суму затримок root (80 мс), TLD (41 мс) і власного сервера (1 мс). За журналом root видно, що після перезапуску резолвер спершу запитав `. IN NS` і `a.root-servers.net IN A`.

**Яке питання залишилося без відповіді.** Чому resolver-a мовчав більше 30 с, а не повертав SERVFAIL, і скільки саме часу він витрачає на спроби до недоступного сервера (рядки `real 0m30.050s` і `0m30.051s` у `t7-queries`). Перевірено: обидва запити дали однаковий результат; після `named-start` відповідь з'явилася лише через 31 с (17:31:52 → 17:32:23). Не з'ясовано: які саме тайм-аути налаштовано в resolver-a (у BIND це, наприклад, `resolver-query-timeout`) — конфігурацію резолвера я не переглядала, а запит з `+timeout` понад 30 с не робила.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Claude Opus 5.5 | Тепер давай ще раз з самого початку бо я вже нічого не розумію | Покрокові інструкції із запуску стенда і команди для завдань 1–7 (зокрема обгортка `r`, команди `sed` для зміни зони, цикли спостереження); команди виконано самостійно на власному стенді |
| 2 | Claude Opus 5.5 | (надіслано файл `all-outputs.txt` з усіма виводами, без тексту) | За моїми виводами ШІ підготував чернетку звіту: заповнені таблиці, відповіді на питання, постмортем і висновки; виводи вставлено з файлів без змін (вилучено лише банер стенда). Чернетку звірено з власними виводами |
