# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ | Кашутіна Юлія Петрівна |
| Група | ЗКН-21 |
| Ідентифікатор | `se-1-14` |
| Дата виконання | 21.11.2026 |
| Середовище | GitHub Codespaces |
| Версія OpenSSL | OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024) |

> Команди виконувалися в терміналі контейнера `se-1-14`, відкритому з термінала codespace командою `docker compose -f stand/compose.yaml exec se-1-14 bash -l` (замість вкладки PORTS 7001), у каталозі `/lab/work`. Для запису виводів використано допоміжну функцію `x(){ echo "$ $*"; eval "$*"; }`, яка друкує команду перед її виконанням, і `tee` у файли `out-*.txt`. Виводи наведено без змін. Час у виводах — UTC.

## Параметри варіанта

| Параметр | Значення |
|---|---|
| Віддалений сервер | `gate.se-1-14.lab` |
| Імена для сертифіката | `www.se-1-14.lab`, `files.se-1-14.lab` |
| Ваш сервер (IP) | `10.53.1.1` |

Стенд під час роботи не перезапускався.

## Завдання 1. HTTP і HTTPS у дампі трафіку

Вивід кроку 1.1:

```text
$ timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 80' > t1-http.txt & sleep 1; curl -s http://gate.se-1-14.lab/; wait
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: gate.se-1-14.lab
SNI: 
protocol: HTTP/1.1 
client: 10.53.1.1
10 packets captured
10 packets received by filter
0 packets dropped by kernel
$ grep -a -E 'GET /|Host:|server: web' t1-http.txt
21:56:07.952248 IP 10.53.1.1.54266 > 10.53.0.80.80: Flags [P.], seq 1:80, ack 1, win 502, options [nop,nop,TS val 451078393 ecr 2900442487], length 79: HTTP: GET / HTTP/1.1
......=wGET / HTTP/1.1
Host: gate.se-1-14.lab
server: web
Host: gate.se-1-14.lab
```

Вивід кроку 1.2:

```text
$ timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t1-https.txt & sleep 1; curl -s --cacert $STAND_CA https://gate.se-1-14.lab/; wait
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: gate.se-1-14.lab
SNI: gate.se-1-14.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1
24 packets captured
24 packets received by filter
0 packets dropped by kernel
$ grep -a -c -E 'GET /|server: web' t1-https.txt
0
$ grep -a -o 'gate.se-1-14.lab' t1-https.txt | wc -l
1
```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` | є (`GET / HTTP/1.1`) | немає (`grep -c` = 0) |
| заголовок `Host` | є (`Host: gate.se-1-14.lab`) | немає |
| тіло відповіді (`server: web`) | є | немає (`grep -c` = 0) |
| ім'я віддаленого сервера (кількість входжень) | 2 (заголовок `Host` у запиті і рядок `Host:` у тілі відповіді) | 1 |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?

   У повідомленні ClientHello, яке йде відкритим текстом, у розширенні SNI (`server_name`). Решта обміну зашифрована, тому ім'я трапляється в дампі лише один раз.

2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?

   SNI — розширення TLS. У запиті по HTTP рукостискання TLS немає, тож і SNI немає (`SNI: `). У HTTPS клієнт передав ім'я в ClientHello, і сервер його показав (`SNI: gate.se-1-14.lab`).

3. Що визначило версію HTTP у другому випадку?

   Узгодження ALPN під час рукостискання TLS: curl запропонував `h2,http/1.1`, сервер обрав `h2`, тому запит пішов як HTTP/2 (`protocol: HTTP/2.0 TLSv1.3`). Для звичайного HTTP без TLS механізму ALPN немає, і curl за замовчуванням використовує HTTP/1.1.

## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```text
$ openssl s_client -connect gate.se-1-14.lab:443 -servername gate.se-1-14.lab -alpn h2,http/1.1 -CAfile $STAND_CA -trace </dev/null > t2-13.txt 2>&1
$ grep -a -E '^(Sent|Received) Record|Content Type|^    [A-Z][A-Za-z]+, Length' t2-13.txt
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=327
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Certificate, Length=981
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    CertificateVerify, Length=75
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Alert (21)
$ grep -a -E '^(ALPN|New,)|Verify return' t2-13.txt
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

Вивід кроку 2.2 (TLS 1.2):

```text
$ openssl s_client -connect gate.se-1-14.lab:443 -servername gate.se-1-14.lab -alpn h2,http/1.1 -CAfile $STAND_CA -trace -tls1_2 </dev/null > t2-12.txt 2>&1
$ grep -a -E '^(Sent|Received) Record|Content Type|^    [A-Z][A-Za-z]+, Length' t2-12.txt
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=222
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
  Content Type = Handshake (22)
    Certificate, Length=976
Received Record
  Content Type = Handshake (22)
    ServerKeyExchange, Length=110
Received Record
  Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
  Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = Handshake (22)
    NewSessionTicket, Length=214
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = ApplicationData (23)
Sent Record
  Content Type = Alert (21)
$ grep -a -E '^(ALPN|New,)|Verify return' t2-12.txt
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
    Verify return code: 0 (ok)
```

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
|---|---|---|---|---|---|
| 1 | К→С | Handshake (22) | — | ClientHello | 327 |
| 2 | С→К | Handshake (22) | — | ServerHello | 118 |
| 3 | С→К | ChangeCipherSpec (20) | — | — | — |
| 4 | С→К | ApplicationData (23) | Handshake (22) | EncryptedExtensions | 15 |
| 5 | С→К | ApplicationData (23) | Handshake (22) | Certificate | 981 |
| 6 | С→К | ApplicationData (23) | Handshake (22) | CertificateVerify | 75 |
| 7 | С→К | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 8 | К→С | ChangeCipherSpec (20) | — | — | — |
| 9 | К→С | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 10 | К→С | ApplicationData (23) | Alert (21) | (close_notify) | — |

Вивід кроків 2.3 і 2.4:

```text
$ grep -a -A3 'extension_type=server_name' t2-13.txt | head -4
        extension_type=server_name(0), length=21
          0000 - 00 13 00 00 10 67 61 74-65 2e 73 65 2d 31 2d   .....gate.se-1-
          000f - 31 34 2e 6c 61 62                              14.lab
        extension_type=ec_point_formats(11), length=4
$ for v in 1.3 1.2; do timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt & sleep 1; curl -s -o /dev/null --tlsv1.2 --tls-max $v --cacert $STAND_CA https://gate.se-1-14.lab/; wait; echo "TLS $v: $(grep -a -c 'OPISM Stand Intermediate' t2-dump-$v.txt)"; done
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
24 packets captured
24 packets received by filter
0 packets dropped by kernel
TLS 1.3: 0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22 packets captured
22 packets received by filter
0 packets dropped by kernel
TLS 1.2: 2
```

| Показник | TLS 1.3 | TLS 1.2 |
|---|---|---|
| Довжина ClientHello, байт | 327 | 222 |
| Номер першого запису, тип якого спостерігач не бачить | 4 (EncryptedExtensions, зовні `ApplicationData (23)`) | немає: у TLS 1.2 зовнішній `Content Type` завжди справжній; зашифровано лише вміст, починаючи із запису 8 (Finished після ChangeCipherSpec) |
| Кількість RTT від ClientHello до першого HTTP-запиту | 1 | 2 |
| Чи видно сертифікат у дампі (результат 2.4) | ні (`TLS 1.3: 0`) | так (`TLS 1.2: 2`) |
| Обраний протокол ALPN | h2 | h2 |

1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?

   Починаючи з четвертого запису з'являється рядок `Inner Content Type = Handshake (22)`, а зовнішній тип стає `Content Type = ApplicationData (23)` — це перший зашифрований запис (EncryptedExtensions). Справжній тип видно лише після розшифрування; спостерігач для всіх наступних записів бачить тільки `ApplicationData (23)` і довжину.

2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.

   У TLS 1.3 клієнт одразу додає розширення `key_share` зі своєю частиною обміну ключами (а також `supported_versions`, `psk_key_exchange_modes`), тому 327 байт проти 222. Саме `key_share` дозволяє серверу в першій же відповіді обчислити спільний ключ і почати шифрування — рукостискання займає 1 RTT замість 2.

3. Довжина розширення `server_name` становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?

   2 байти — довжина списку імен, 1 байт — тип імені (`00` = host_name), 2 байти — довжина імені, далі саме ім'я: 2 + 1 + 2 + 12 = 17. Для `gate.se-1-14.lab` (16 символів): 2 + 1 + 2 + 16 = 21, що збігається з виводом `extension_type=server_name(0), length=21` (байти `00 13` = 19 — довжина списку, `00` — тип, `00 10` = 16 — довжина імені).

4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?

   Це режим сумісності з проміжним обладнанням (middlebox compatibility mode): старі брандмауери й проксі, які знають лише TLS 1.2, очікують ChangeCipherSpec перед зашифрованими даними і можуть обривати з'єднання без нього. Тому TLS 1.3 «маскується» під відновлену сесію TLS 1.2.

## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```text
$ acme-root
збережено /lab/tls/acme-root.crt
$ openssl x509 -in /lab/tls/acme-root.crt -noout -subject -dates
subject=CN = Pebble Root CA 10ca2b
notBefore=Nov 21 21:52:25 2026 GMT
notAfter=Nov 21 21:52:25 2056 GMT
$ curl -sS --cacert /lab/tls/acme-root.crt https://www.se-1-14.lab/
curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Вивід кроку 3.3 (вивід `lego` повністю і рядки журналу nginx):

```text
$ lego --server $ACME_DIR --email admin@se-1-14.lab --accept-tos --path /lab/work/lego --http --http.webroot /var/www/acme --domains www.se-1-14.lab --domains files.se-1-14.lab run --no-bundle
2026/11/21 22:09:49 No key found for account admin@se-1-14.lab. Generating a P256 key.
2026/11/21 22:09:49 Saved key to /lab/work/lego/accounts/ca.lab_14000/admin@se-1-14.lab/keys/admin@se-1-14.lab.key
2026/11/21 22:09:49 [INFO] acme: Registering account for admin@se-1-14.lab
!!!! HEADS UP !!!!

Your account credentials have been saved in your Let's Encrypt
configuration directory at "/lab/work/lego/accounts".

You should make a secure backup of this folder now. This
configuration directory will also contain certificates and
private keys obtained from Let's Encrypt so making regular
backups of this folder is ideal.
2026/11/21 22:09:49 [INFO] [www.se-1-14.lab, files.se-1-14.lab] acme: Obtaining SAN certificate
2026/11/21 22:09:50 [INFO] [files.se-1-14.lab] AuthURL: https://ca.lab:14000/authZ/YJN-QEPXY-BKCklWED1vCV9ZMZH_bIaLwLMOnCCQH6w
2026/11/21 22:09:50 [INFO] [www.se-1-14.lab] AuthURL: https://ca.lab:14000/authZ/VUce2DfGlrYb5A9IkCLxPAh5aUUH8l7G7SH6XteMgbA
2026/11/21 22:09:50 [INFO] [www.se-1-14.lab] acme: Could not find solver for: tls-alpn-01
2026/11/21 22:09:50 [INFO] [www.se-1-14.lab] acme: use http-01 solver
2026/11/21 22:09:50 [INFO] [files.se-1-14.lab] acme: Could not find solver for: tls-alpn-01
2026/11/21 22:09:50 [INFO] [files.se-1-14.lab] acme: use http-01 solver
2026/11/21 22:09:50 [INFO] [www.se-1-14.lab] acme: Trying to solve HTTP-01
2026/11/21 22:09:53 [INFO] [www.se-1-14.lab] The server validated our request
2026/11/21 22:09:53 [INFO] [files.se-1-14.lab] acme: Trying to solve HTTP-01
2026/11/21 22:09:56 [INFO] [files.se-1-14.lab] The server validated our request
2026/11/21 22:09:56 [INFO] [www.se-1-14.lab, files.se-1-14.lab] acme: Validations succeeded; requesting certificates
2026/11/21 22:09:56 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/21 22:09:56 [INFO] [www.se-1-14.lab] Server responded with a certificate.
$ grep acme-challenge /var/log/nginx/access.log
10.53.0.10 - - [21/Nov/2026:22:09:50 +0000] "GET /.well-known/acme-challenge/BVKx8MdlNLEgVSz7Tnqa-N3Q2n0OttBdBJYxr30MgbE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [21/Nov/2026:22:09:50 +0000] "GET /.well-known/acme-challenge/BVKx8MdlNLEgVSz7Tnqa-N3Q2n0OttBdBJYxr30MgbE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [21/Nov/2026:22:09:50 +0000] "GET /.well-known/acme-challenge/BVKx8MdlNLEgVSz7Tnqa-N3Q2n0OttBdBJYxr30MgbE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [21/Nov/2026:22:09:53 +0000] "GET /.well-known/acme-challenge/JzUf4H0jAaFcujFlpknC9usREoy3I2PLz5r1LoDWLEA HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [21/Nov/2026:22:09:53 +0000] "GET /.well-known/acme-challenge/JzUf4H0jAaFcujFlpknC9usREoy3I2PLz5r1LoDWLEA HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [21/Nov/2026:22:09:53 +0000] "GET /.well-known/acme-challenge/JzUf4H0jAaFcujFlpknC9usREoy3I2PLz5r1LoDWLEA HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
```

Вивід кроку 3.4:

```text
$ openssl x509 -in www.se-1-14.lab.crt -noout -subject -issuer -dates -serial -ext subjectAltName
subject=CN = www.se-1-14.lab
issuer=CN = Pebble Intermediate CA 771916
notBefore=Nov 21 22:09:56 2026 GMT
notAfter=Nov 21 22:09:55 2031 GMT
serial=474592156A59C2F8
X509v3 Subject Alternative Name: 
    DNS:www.se-1-14.lab, DNS:files.se-1-14.lab
$ openssl x509 -in www.se-1-14.lab.issuer.crt -noout -subject -issuer
subject=CN = Pebble Intermediate CA 771916
issuer=CN = Pebble Root CA 10ca2b
```

Вивід кроку 3.5:

```text
$ cp www.se-1-14.lab.crt /lab/tls/server.crt
$ cp www.se-1-14.lab.key /lab/tls/server.key
$ web-reload
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/21 22:10:58 [notice] 113#113: signal process started
$ curl -sS --cacert /lab/tls/acme-root.crt https://www.se-1-14.lab/
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
$ openssl s_client -connect www.se-1-14.lab:443 -servername www.se-1-14.lab -CAfile /lab/tls/acme-root.crt </dev/null 2>/dev/null | sed -n '/^Certificate chain/,/^---/p;/^Verify return code/p'
Certificate chain
 0 s:CN = www.se-1-14.lab
   i:CN = Pebble Intermediate CA 771916
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 21 22:09:56 2026 GMT; NotAfter: Nov 21 22:09:55 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)
```

Вивід кроку 3.6:

```text
$ cat www.se-1-14.lab.crt www.se-1-14.lab.issuer.crt > /lab/tls/server.crt
$ web-reload
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/21 22:10:58 [notice] 123#123: signal process started
$ curl -sS --cacert /lab/tls/acme-root.crt https://www.se-1-14.lab/
server: se-1-14 (https)
Host: www.se-1-14.lab
SNI: www.se-1-14.lab
protocol: TLSv1.3
$ openssl s_client -connect www.se-1-14.lab:443 -servername www.se-1-14.lab -CAfile /lab/tls/acme-root.crt </dev/null 2>/dev/null | sed -n '/^Certificate chain/,/^---/p;/^Verify return code/p'
Certificate chain
 0 s:CN = www.se-1-14.lab
   i:CN = Pebble Intermediate CA 771916
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 21 22:09:56 2026 GMT; NotAfter: Nov 21 22:09:55 2031 GMT
 1 s:CN = Pebble Intermediate CA 771916
   i:CN = Pebble Root CA 10ca2b
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 21 21:52:25 2026 GMT; NotAfter: Nov 21 21:52:25 2056 GMT
---
Verify return code: 0 (ok)
$ curl -sS --cacert /lab/tls/acme-root.crt https://files.se-1-14.lab/
server: se-1-14 (https)
Host: files.se-1-14.lab
SNI: files.se-1-14.lab
protocol: TLSv1.3
```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера | `CN = Pebble Intermediate CA 771916` |
| Видавець сертифіката видавця | `CN = Pebble Root CA 10ca2b` |
| Імена в SAN | `www.se-1-14.lab`, `files.se-1-14.lab` |
| Строк дії (від – до) | Nov 21 22:09:56 2026 GMT – Nov 21 22:09:55 2031 GMT |
| Тип ключа сервера і алгоритм підпису | EC P-256 (`id-ecPublicKey, 256 (bit)`), підпис `RSA-SHA256` |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 | 1 і 2 |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.

   Центр сертифікації Pebble (його валідатор, адреса 10.53.0.10) виконав перевірку HTTP-01: `lego` поклав на мій сервер файли-токени, а валідатор завантажив їх за адресою `http://<ім'я>/.well-known/acme-challenge/<токен>` для кожного імені. Підтвердження: рядки `lego` `[www.se-1-14.lab] The server validated our request` і `[files.se-1-14.lab] The server validated our request`, а в журналі nginx — по три запити `GET /.well-known/acme-challenge/...` з кодом 200 від `10.53.0.10` з агентом `LetsEncrypt-Pebble-VA` для двох різних токенів.

2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?

   Сервер надіслав лише свій сертифікат (у ланцюжку тільки `0 s:CN = www.se-1-14.lab`), а підписав його проміжний центр `Pebble Intermediate CA 771916`. Клієнт має лише корінь і не може побудувати ланцюжок «сервер → проміжний → корінь», бо проміжного сертифіката немає: `curl: (60) ... unable to get local issuer certificate`, `Verify return code: 21 (unable to verify the first certificate)`.

3. Що означає `id-ecPublicKey` і `RSA-SHA256` і чому вони не суперечать одне одному?

   `id-ecPublicKey, 256 (bit)` — тип відкритого ключа самого сервера, що міститься в сертифікаті (еліптична крива P-256; `lego` згенерував EC-ключ). `sigalg: RSA-SHA256` — алгоритм, яким видавець підписав цей сертифікат: проміжний центр має ключ RSA 2048 (`a:PKEY: rsaEncryption, 2048 (bit)` у рядку 1 ланцюжка). Це ключі різних власників, тому їхні алгоритми можуть відрізнятися.

4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?

   Pebble під час кожного запуску створює нові кореневий і проміжний ключі, тому після перезапуску новий корінь не підписував мій сертифікат: перевірка з новим `acme-root.crt` не пройде (`unable to get local issuer certificate`), і сертифікат доведеться видати повторно. У реальності це відповідає заміні або відкликанню кореня чи проміжного центру (наприклад, через компрометацію або вилучення центру зі сховищ довіри) — сервери мусять перевипустити сертифікати в нового центру.

## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```text
$ for i in 1 2 3 4 5; do n=f$i.se-1-14.lab; echo "=== $n"; curl -sS --cacert $STAND_CA https://$n/ 2>&1 | head -1; openssl s_client -connect $n:443 -servername $n -CAfile $STAND_CA </dev/null 2>/dev/null | sed -n "/^Certificate chain/,/^---/p;/^Verify return code/p" | grep -v PKEY; done
=== f1.se-1-14.lab
curl: (60) SSL certificate problem: self-signed certificate
Certificate chain
 0 s:CN = f1.se-1-14.lab
   i:CN = f1.se-1-14.lab
   v:NotBefore: Nov 21 21:52:23 2026 GMT; NotAfter: Feb 19 21:52:23 2027 GMT
---
Verify return code: 18 (self-signed certificate)
=== f2.se-1-14.lab
server: web, f2
Certificate chain
 0 s:CN = f2.se-1-14.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 21 21:52:23 2026 GMT; NotAfter: Feb 19 21:52:23 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 21 21:52:23 2026 GMT; NotAfter: Nov 20 21:52:23 2031 GMT
---
Verify return code: 0 (ok)
=== f3.se-1-14.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f3.se-1-14.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 21 21:52:24 2026 GMT; NotAfter: Feb 19 21:52:24 2027 GMT
---
Verify return code: 21 (unable to verify the first certificate)
=== f4.se-1-14.lab
curl: (60) SSL: no alternative certificate subject name matches target host name 'f4.se-1-14.lab'
Certificate chain
 0 s:CN = legacy.se-1-14.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 21 21:52:24 2026 GMT; NotAfter: Feb 19 21:52:24 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 21 21:52:23 2026 GMT; NotAfter: Nov 20 21:52:23 2031 GMT
---
Verify return code: 0 (ok)
=== f5.se-1-14.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f5.se-1-14.lab
   i:O = Foreign Org, CN = Foreign Intermediate CA
   v:NotBefore: Nov 21 21:52:24 2026 GMT; NotAfter: Feb 19 21:52:24 2027 GMT
 1 s:O = Foreign Org, CN = Foreign Intermediate CA
   i:O = Foreign Org, CN = Foreign Root CA
   v:NotBefore: Nov 21 21:52:23 2026 GMT; NotAfter: Nov 20 21:52:23 2031 GMT
---
Verify return code: 20 (unable to get local issuer certificate)
```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 | `SSL certificate problem: self-signed certificate` | 18 (self-signed certificate) | самопідписаний сертифікат: ланцюжок не веде до довіреного кореня | `0 s:CN = f1.se-1-14.lab` / `i:CN = f1.se-1-14.lab` (видавець = власник) | отримати сертифікат від довіреного центру сертифікації (наприклад, через ACME, як у завданні 3) |
| f2 | — (відповідь `server: web, f2`) | 0 (ok) | усі перевірки пройдено, сервер справний | `Verify return code: 0 (ok)`, ланцюжок із 2 сертифікатів до OPISM Stand Root CA | нічого |
| f3 | `unable to get local issuer certificate` | 21 (unable to verify the first certificate) | сервер не надсилає проміжний сертифікат | у ланцюжку лише `0 s:CN = f3.se-1-14.lab`, `i:... OPISM Stand Intermediate CA`, рядка `1` немає | додати сертифікат проміжного центру у файл сертифіката сервера після сертифіката сервера (як у кроці 3.6) |
| f4 | `no alternative certificate subject name matches target host name 'f4.se-1-14.lab'` | 0 (ok) | ім'я відсутнє в SAN | `0 s:CN = legacy.se-1-14.lab` — сертифікат виданий для іншого імені | перевипустити сертифікат з іменем `f4.se-1-14.lab` у SAN (або встановити правильний сертифікат) |
| f5 | `unable to get local issuer certificate` | 20 (unable to get local issuer certificate) | сертифікат виданий центром, якому клієнт не довіряє | `i:O = Foreign Org, CN = Foreign Intermediate CA` → `i:O = Foreign Org, CN = Foreign Root CA` | отримати сертифікат у центру, якому довіряють клієнти (або клієнти мають додати Foreign Root CA до довірених) |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність. Чим ці дві несправності відрізняються у виводі `openssl s_client`?

   Цю помилку дали два сервери. f3 не надсилає проміжний сертифікат: у ланцюжку лише сертифікат `0`, а код — `21 (unable to verify the first certificate)`. f5 надсилає повний ланцюжок (`0` і `1`), але він веде до кореня `Foreign Root CA`, якого немає серед довірених: код `20 (unable to get local issuer certificate)`. Тобто відмінність — у кількості сертифікатів у ланцюжку, у видавці і в коді 21 чи 20.

2. Чому `openssl s_client` повертає для f4 `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку?

   `openssl s_client` без додаткових ключів (`-verify_hostname`) перевіряє лише ланцюжок підписів до довіреного кореня, а він для f4 правильний. `curl` додатково звіряє ім'я, до якого підключається, з іменами в сертифікаті; `f4.se-1-14.lab` там немає (є лише `legacy.se-1-14.lab`), тому помилка.

3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?

   Усі п'ять: рукостискання TLS з кожним сервером відбувається, а `-k` вимикає всі перевірки сертифіката (строк, ім'я, ланцюжок довіри). Шифрування залишається, але втрачається автентифікація сервера: клієнт не знає, з ким говорить, і з'єднання можна перехопити атакою «людина посередині».

## Завдання 5. Ціна нового з'єднання

**Прогноз** (записано до вимірювання). Скільки RTT мине до завершення кожного етапу (від початку запиту, з урахуванням 1 RTT на пошук імені):

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | 2 (DNS + TCP) | 2 | 0 |
| `time_appconnect` | 3 (+1 RTT TLS 1.3) | 4 (+2 RTT TLS 1.2) | 0 |
| `time_starttransfer` | 4 | 5 | 1 |

Вивід кроків 5.2 і 5.3:

```text
$ for i in 1 2 3; do curl -s -o /dev/null --cacert $STAND_CA -w "$F" $U; done
0.060964 0.091132 0.124556 0.155002 0.155056 2 1
0.030637 0.060794 0.094084 0.124645 0.124694 2 1
0.030749 0.060882 0.094305 0.124641 0.124695 2 1
$ for i in 1 2 3; do curl -s -o /dev/null --tls-max 1.2 --cacert $STAND_CA -w "$F" $U; done
0.030642 0.060795 0.124277 0.154696 0.154753 2 1
0.030615 0.060750 0.124029 0.154427 0.154517 2 1
0.030656 0.060804 0.125427 0.155887 0.155958 2 1
$ curl -s --cacert $STAND_CA -w "$F" -o /dev/null $U -o /dev/null $U -o /dev/null $U
0.030599 0.060936 0.096594 0.127198 0.127256 2 1
0.000017 0.000000 0.000000 0.030591 0.030643 2 0
0.000017 0.000000 0.000000 0.030231 0.030266 2 0
```

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове | 30.7 | 60.9 | 94.3 | 124.6 | 124.7 | 1 |
| TLS 1.2, нове | 30.6 | 60.8 | 124.3 | 154.7 | 154.8 | 1 |
| 5.3, перший запит | 30.6 | 60.9 | 96.6 | 127.2 | 127.3 | 1 |
| 5.3, другий запит | 0.017 | 0 | 0 | 30.6 | 30.6 | 0 |
| 5.3, третій запит | 0.017 | 0 | 0 | 30.2 | 30.3 | 0 |

RTT, визначений вимірюванням, і з якого значення його визначено: **RTT ≈ 30 мс**. Визначено з різниці `time_connect − time_namelookup` (60.9 − 30.7 = 30.2 мс — встановлення TCP займає рівно 1 RTT), а також з `time_starttransfer` повторних запитів у 5.3 (30.6 і 30.2 мс — один «запит–відповідь» без нового з'єднання).

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?

   TLS 1.3: `namelookup` ≈ 1 RTT, `connect` ≈ 2, `appconnect` ≈ 3.1, `starttransfer` ≈ 4.2. TLS 1.2: 1, 2, 4.1, 5.2. Повторне з'єднання: `starttransfer` ≈ 1 RTT. Результат збігся з прогнозом. Відхилення від цілих (приблизно +3–4 мс на `appconnect` і далі) — це час обчислень: обмін ключами, перевірка підпису і ланцюжка сертифікатів, обробка запиту сервером. Перший вимір TLS 1.3 має `namelookup` 61 мс (2 RTT) — див. питання 2.

2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?

   Резолвер — окремий вузол (10.53.0.53), і запит до нього проходить через вихідний інтерфейс мого контейнера, на якому стенд додає затримку, тому навіть відповідь з кешу коштує 1 RTT (~30 мс). Удвічі більшим (2 RTT) він був би, якби резолвер не мав імені в кеші й мусив сам спитати авторитативний сервер зони (10.53.1.1 — мій контейнер, тобто ще один прохід через затримку), або якби клієнт робив запити A і AAAA послідовно. Саме так виглядає перший вимір: `0.060964` с.

3. Сервіс виконує 200 послідовних запитів до цього сервера. Скільки часу займе лише очікування мережі?

   - Нове з'єднання TLS 1.3 на кожен запит: ≈ 4 RTT × 200 = 800 RTT ≈ 800 × 30 мс = **24 с** (за вимірами 200 × 124.7 мс ≈ 24.9 с).
   - Нове з'єднання TLS 1.2: ≈ 5 RTT × 200 = 1000 RTT ≈ **30 с** (за вимірами 200 × 154.8 мс ≈ 31 с).
   - Одне з'єднання на всі запити: 4 RTT на перший + 199 × 1 RTT = 203 RTT ≈ **6.1 с** (за вимірами 127.3 мс + 199 × 30.5 мс ≈ 6.2 с).

## Висновки

**Що виявилося несподіваним.** Сервер f4 з сертифікатом для чужого імені (`0 s:CN = legacy.se-1-14.lab`) отримав від `openssl s_client` `Verify return code: 0 (ok)` — я очікувала, що будь-який інструмент відхилить такий сертифікат, а помилку показав лише `curl` (`no alternative certificate subject name matches target host name 'f4.se-1-14.lab'`). Також несподівано, що в TLS 1.3 сертифікат сервера в дампі не видно взагалі (`TLS 1.3: 0`), а в TLS 1.2 видно (`TLS 1.2: 2`), хоча обидва з'єднання захищені.

**Яке питання залишилося без відповіді.** Чому перший вимір у кроці 5.2 має `time_namelookup` 0.060964 с (2 RTT), а всі наступні — близько 0.0306 с (1 RTT). Моя гіпотеза — запис `gate.se-1-14.lab` на той момент вийшов з кешу резолвера, і резолвер мусив звернутися до авторитативного сервера; але TTL цього запису і журнали резолвера я не перевіряла, тому підтвердити гіпотезу не вдалося.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Claude Opus 5.5 | Покрокові інструкції із запуску стенда і команди для завдань 1–5 (зокрема обгортка `x` і запис виводів у файли через `tee`); команди виконано самостійно на власному стенді |
| 2 | Claude Opus 5.5 | (надіслано файл `all-tls.txt` з усіма виводами, без тексту) | За моїми виводами ШІ підготував чернетку звіту: таблиці, відповіді на питання, висновки. Чернетку звірено з власними виводами |
