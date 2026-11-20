# Практична робота № 1

**Дисципліна:** Основи побудови інформаційних систем та мереж

**Тема:** Спостереження за процесом звернення до вебресурсу. Побудова власної моделі рівнів взаємодії

| | |
|---|---|
| **Прізвище, ім'я** |Юлія Кашутіна |
| **Група** |ЗКН - 21 |
| **Номер варіанта** |14 |
| **Домен варіанта** |freebsd.org |
| **Середовище виконання** | macOS |
| **Версія curl** | curl 8.7.1 (x86_64-apple-darwin25.0) libcurl/8.7.1 (SecureTransport) LibreSSL/3.3.6 zlib/1.2.12 nghttp2/1.68.0 |
| **Дата виконання** |20.11.2026 |

---

## Частина A. Збір експериментальних даних

### A.1. Запит із діагностичним виводом

**Команда:**

```
curl -v https://freebsd.org
```

**Вивід:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host freebsd.org:443 was resolved.
* IPv6: (none)
* IPv4: 96.47.72.84
*   Trying 96.47.72.84:443...
* Connected to freebsd.org (96.47.72.84) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
} [316 bytes data]
*  CAfile: /etc/ssl/cert.pem
*  CApath: none

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* (304) (IN), TLS handshake, Server hello (2):
{ [122 bytes data]
* (304) (IN), TLS handshake, Unknown (8):
{ [25 bytes data]
* (304) (IN), TLS handshake, Certificate (11):
{ [3480 bytes data]
* (304) (IN), TLS handshake, CERT verify (15):
{ [79 bytes data]
* (304) (IN), TLS handshake, Finished (20):
{ [52 bytes data]
* (304) (OUT), TLS handshake, Finished (20):
} [52 bytes data]
* SSL connection using TLSv1.3 / AEAD-AES256-GCM-SHA384 / [blank] / UNDEF
* ALPN: server accepted http/1.1
* Server certificate:
*  subject: CN=www.freebsd.org
*  start date: Nov 10 20:54:00 2026 GMT
*  expire date: Feb  8 20:53:59 2027 GMT
*  subjectAltName: host "freebsd.org" matched cert's "freebsd.org"
*  issuer: C=US; O=Let's Encrypt; CN=YE1
*  SSL certificate verify ok.
* using HTTP/1.x
> GET / HTTP/1.1
> Host: freebsd.org
> User-Agent: curl/8.7.1
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Server: nginx
< Content-Type: text/html
< Content-Length: 162
< Connection: keep-alive
< Location: https://www.freebsd.org/
< Strict-Transport-Security: max-age=31536000
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< X-Frame-Options: SAMEORIGIN
< 
{ [162 bytes data]

100   162  100   162    0     0    711      0 --:--:-- --:--:-- --:--:--   710
* Connection #0 to host freebsd.org left intact
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

---

### A.2. Запит без захисту з'єднання

**Команда:**

```
curl -v http://neverssl.com
```

**Вивід:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host neverssl.com:80 was resolved.
* IPv6: (none)
* IPv4: 34.223.124.45
*   Trying 34.223.124.45:80...

  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:02 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:03 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:04 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:06 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:07 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:08 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:09 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:11 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:12 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:13 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:14 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:15 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:16 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:17 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:18 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:19 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:20 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:21 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:22 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:23 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:24 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:25 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:26 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:27 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:28 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:29 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:30 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:31 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:32 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:33 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:34 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:35 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:36 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:37 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:38 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:39 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:40 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:41 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:42 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:43 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:44 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:45 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:46 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:47 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:48 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:49 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:50 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:51 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:52 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:53 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:54 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:55 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:56 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:57 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:58 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:59 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:00 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:01 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:02 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:03 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:04 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:05 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:06 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:07 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:08 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:09 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:10 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:11 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:12 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:13 --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:01:14 --:--:--     0* connect to 34.223.124.45 port 80 from 10.0.0.72 port 53646 failed: Operation timed out
* Failed to connect to neverssl.com port 80 after 75047 ms: Couldn't connect to server

  0     0    0     0    0     0      0      0 --:--:--  0:01:15 --:--:--     0
* Closing connection
curl: (28) Failed to connect to neverssl.com port 80 after 75047 ms: Couldn't connect to server
```

---

### A.3. Запит до служби доменних імен

**Команда (перше виконання):**

```
dig freebsd.org
```

**Вивід:**

```
12:08

; <<>> DiG 9.10.6 <<>> freebsd.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17268
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;freebsd.org.			IN	A

;; ANSWER SECTION:
freebsd.org.		3600	IN	A	96.47.72.84

;; Query time: 265 msec
;; SERVER: 24.200.243.189#53(24.200.243.189)
;; WHEN: Fri Nov 20 12:08:09 EST 2026
;; MSG SIZE  rcvd: 56
```

**Команда (повторне виконання через 5–7 хвилин):**

```
dig freebsd.org
```

**Вивід:**

```
12:13

; <<>> DiG 9.10.6 <<>> freebsd.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38814
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;freebsd.org.			IN	A

;; ANSWER SECTION:
freebsd.org.		3600	IN	A	96.47.72.84

;; Query time: 28 msec
;; SERVER: 24.200.243.189#53(24.200.243.189)
;; WHEN: Fri Nov 20 12:13:04 EST 2026
;; MSG SIZE  rcvd: 56
```

**Зафіксовані значення:**

| Параметр | Перше виконання | Повторне виконання |
|---|---|---|
| Час виконання (год:хв) | 12:08 | 12:13 |
| IP-адреса | 96.47.72.84 | 96.47.72.84 |
| Значення TTL | 3600 | 3600 |

> Якщо друге значення TTL виявилося більшим за перше — це нормально: кеш резолвера встиг оновитися. Зафіксуйте як є.

---

### A.4. Контрольний ресурс

**Команда:**

```
curl -v https://google.com
```

**Вивід:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host google.com:443 was resolved.
* IPv6: (none)
* IPv4: 142.250.69.142
*   Trying 142.250.69.142:443...
* Connected to google.com (142.250.69.142) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
} [315 bytes data]
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
{ [122 bytes data]
* (304) (IN), TLS handshake, Unknown (8):
{ [15 bytes data]
* (304) (IN), TLS handshake, Certificate (11):
{ [3561 bytes data]
* (304) (IN), TLS handshake, CERT verify (15):
{ [79 bytes data]
* (304) (IN), TLS handshake, Finished (20):
{ [36 bytes data]
* (304) (OUT), TLS handshake, Finished (20):
} [36 bytes data]
* SSL connection using TLSv1.3 / AEAD-CHACHA20-POLY1305-SHA256 / [blank] / UNDEF
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=*.google.com
*  start date: Nov  3 08:26:50 2026 GMT
*  expire date: Jan 26 08:26:49 2027 GMT
*  subjectAltName: host "google.com" matched cert's "google.com"
*  issuer: C=US; O=Google Trust Services; CN=WE2
*  SSL certificate verify ok.
* using HTTP/2
* [HTTP/2] [1] OPENED stream for https://google.com/
* [HTTP/2] [1] [:method: GET]
* [HTTP/2] [1] [:scheme: https]
* [HTTP/2] [1] [:authority: google.com]
* [HTTP/2] [1] [:path: /]
* [HTTP/2] [1] [user-agent: curl/8.7.1]
* [HTTP/2] [1] [accept: */*]
> GET / HTTP/2
> Host: google.com
> User-Agent: curl/8.7.1
> Accept: */*
> 
* Request completely sent off
< HTTP/2 301 
< location: https://www.google.com/
< content-type: text/html; charset=UTF-8
< content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-CJnNj3kgntlBOqrqi4GHaQ' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
< date: Fri, 20 Nov 2026 16:33:23 GMT
< expires: Sun, 20 Dec 2026 16:33:23 GMT
< cache-control: public, max-age=2592000
< server: gws
< content-length: 220
< x-xss-protection: 0
< x-frame-options: SAMEORIGIN
< alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
< 
{ [220 bytes data]

100   220  100   220    0     0   1616      0 --:--:-- --:--:-- --:--:--  1617
* Connection #0 to host google.com left intact
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

---

### A.5. Ресурси з некоректною конфігурацією сертифіката

**Випадок 1**

```
curl -v https://expired.badssl.com
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host expired.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* Connected to expired.badssl.com (104.154.89.105) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
} [323 bytes data]
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
{ [108 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [4323 bytes data]
* TLSv1.2 (OUT), TLS alert, certificate expired (557):
} [2 bytes data]
* SSL certificate problem: certificate has expired

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [4323 bytes data]
* TLSv1.2 (OUT), TLS alert, certificate expired (557):
} [2 bytes data]
curl: (60) SSL certificate problem: certificate has expired
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

**Випадок 2**

```
curl -v https://wrong.host.badssl.com
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host wrong.host.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* Connected to wrong.host.badssl.com (104.154.89.105) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
} [326 bytes data]
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
{ [108 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [4074 bytes data]
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
{ [333 bytes data]
* TLSv1.2 (IN), TLS handshake, Server finished (14):
{ [4 bytes data]
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
} [70 bytes data]
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.2 (OUT), TLS handshake, Finished (20):
} [16 bytes data]
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
{ [1 bytes data]
* TLSv1.2 (IN), TLS handshake, Finished (20):
{ [16 bytes data]
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256 / [blank] / UNDEF
* ALPN: server accepted http/1.1
* Server certificate:
*  subject: CN=*.badssl.com
*  start date: Sep 29 20:02:56 2026 GMT
*  expire date: Dec 28 20:02:55 2026 GMT
*  subjectAltName does not match host name wrong.host.badssl.com
* SSL: no alternative certificate subject name matches target host name 'wrong.host.badssl.com'

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection
* TLSv1.2 (OUT), TLS alert, close notify (256):
} [2 bytes data]
curl: (60) SSL: no alternative certificate subject name matches target host name 'wrong.host.badssl.com'
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

**Випадок 3**

```
curl -v https://self-signed.badssl.com
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host self-signed.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* Connected to self-signed.badssl.com (104.154.89.105) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
} [327 bytes data]
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
{ [108 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [903 bytes data]
* TLSv1.2 (OUT), TLS alert, unknown CA (560):
} [2 bytes data]
* SSL certificate problem: self signed certificate

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [903 bytes data]
* TLSv1.2 (OUT), TLS alert, unknown CA (560):
} [2 bytes data]
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

> Якщо використано альтернативний спосіб із параметром `--resolve` — зазначити це та навести фактичну команду.

---

## Частина B. Власна модель рівнів

**Кількість виділених груп:** 5

| № | Назва групи (власне формулювання) | Рядки виводу, віднесені до групи | Обґрунтування |
|---|---|---|---|
| 1 |Вміст сторінки |`<html>` … `</html>` (A.1) |Те, що отримує і бачить користувач; з'являється останнім |
| 2 | Запит і відповідь | `> GET / HTTP/1.1`, `> Host: freebsd.org`, `< HTTP/1.1 301 Moved Permanently`, `< Location: https://www.freebsd.org/` | Клієнт просить конкретний ресурс, сервер відповідає кодом і заголовками |
| 3 | Захист каналу | `TLS handshake, Client hello (1)` … `Finished (20)`, `SSL connection using TLSv1.3`, `Server certificate`, `SSL certificate verify ok.` | Відбувається до запиту; у незахищеному A.2 цього блоку немає зовсім |
| 4 | З'єднання з програмою на сервері | `Trying 96.47.72.84:443...`, `Connected to freebsd.org (96.47.72.84) port 443` | Встановлюється зв'язок з конкретним портом; в A.2 саме тут стався збій (`Operation timed out`) |
| 5 | Пошук адреси за іменем | `Host freebsd.org:443 was resolved.`, `IPv4: 96.47.72.84`, вивід `dig freebsd.org` | Без IP-адреси жоден інший етап неможливий; виконується першим |

*Групи впорядковано від найближчої до користувача (№ 1) до найближчої до апаратного забезпечення. Зайві рядки вилучити, за потреби — додати.*

**Рядки, які не вдалося віднести до жодної групи:**

| Рядок виводу | Причина утруднення |
|---|---|
| `% Total    % Received % Xferd  Average Speed ...` | Це лічильник прогресу самої програми curl, а не етап обміну з сервером |
| `{ [122 bytes data]` | Показує лише обсяг переданих даних, без змісту — незрозуміло, до якого етапу належить |
| `* Connection #0 to host freebsd.org left intact` | Стосується і з'єднання (група 4), і завершення обміну (група 2) |

---

## Контрольні питання

**1. Скільки рядків діагностичного виводу передує отриманню даних сторінки (завдання A.1)?**

> 51 рядок. Рахувала рядки, що починаються з `*`, `>`, `<`, `{`, `}`, від `Host freebsd.org:443 was resolved.` до `Connection #0 to host freebsd.org left intact` включно. Рядки лічильника прогресу curl (`% Total ...`, `0 0 0 ...`) не рахувала. До того ж «даними сторінки» виявилася не сама сторінка, а коротка відповідь `301 Moved Permanently`.

**2. Які рядки наявні у виводі A.1 і відсутні у виводі A.2? Чим це зумовлено?**

> В A.2 відсутні: рядок `Connected to ...`, усі рядки `TLS handshake`, блок `Server certificate` (subject, start/expire date, issuer, `SSL certificate verify ok.`), а також рядки запиту `>` та відповіді `<`. Блоку TLS немає, бо адреса починається з `http://` — це незахищене з'єднання без шифрування і перевірки сертифіката. Рядків `>` і `<` немає, бо в моєму випадку з'єднання з портом 80 взагалі не встановилося: `connect to 34.223.124.45 port 80 ... failed: Operation timed out` через 75047 мс, тому запит не був надісланий.

**3. Звідки у виводі з'явилося значення `443`, якщо його не було вказано в адресі?**

> В адресі вказано схему `https`, а стандартний порт для неї — 443. curl підставив його автоматично, що видно з рядків `Host freebsd.org:443 was resolved.` і `Connected to freebsd.org (96.47.72.84) port 443`. Для `http://neverssl.com` так само автоматично підставився порт 80.

**4. Як змінилося значення TTL між двома запитами (A.3)? Що означає це число?**

> Не змінилося: о 12:08 і о 12:13 TTL дорівнював 3600. TTL — це кількість секунд, протягом яких резолвер може зберігати запис у кеші й не питати повторно. Якби друга відповідь бралася з того самого кешу, через 5 хвилин TTL мав би зменшитися приблизно до 3300. Те, що він знову 3600, означає, що резолвер віддав повне значення — ймовірно, отримав запис заново або відповідав інший сервер резолвера. Водночас час запиту зменшився з 265 мс до 28 мс.

**5. Чим відрізняються між собою три причини помилок із завдання A.5? Сформулювати кожну однією фразою.**

| Випадок | Причина недовіри |
|---|---|
| `expired` | Строк дії сертифіката вже минув (`certificate has expired`). |
| `wrong.host` | Сертифікат видано для `*.badssl.com`, і він не підходить для імені `wrong.host.badssl.com` (`no alternative certificate subject name matches`). |
| `self-signed` | Сертифікат підписав сам власник сервера, а не довірений центр сертифікації (`self signed certificate`, `unknown CA`). |

**6. Три рядки з власних виводів, про які не йшлося на лекції 1:**

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `* ALPN: curl offers h2,http/1.1` | A.1 |
| 2 | `< Strict-Transport-Security: max-age=31536000` | A.1 |
| 3 | `* TLSv1.2 (OUT), TLS alert, certificate expired (557):` | A.5 |

*Пояснення до цих рядків не потрібне.*

---

## Висновки

*150–300 слів. Спиратися на власні спостереження, а не на матеріал лекції.*

**D.1. Що виявилося неочевидним або несподіваним**

*Назвати конкретно, з посиланням на рядок виводу.*

> Я очікувала отримати сторінку freebsd.org, але сервер відповів `HTTP/1.1 301 Moved Permanently` з `Location: https://www.freebsd.org/` — тобто замість вмісту надіслав нову адресу. Так само вчинив google.com (`HTTP/2 301`). Несподіваним було й те, що запит A.2 до neverssl.com не відбувся взагалі: `Operation timed out` після 75047 мс, хоча ім'я успішно перетворилося на адресу 34.223.124.45. Також помітила, що freebsd.org використовує TLSv1.3, а сервери badssl.com — TLSv1.2, і в них рукостискання обривається одразу після отримання сертифіката (`TLS alert, certificate expired`).

**D.2. Чому саме така кількість груп у частині B**

*На якій підставі ухвалено рішення. Що змусило б його змінити.*

> Я виділила 5 груп, бо у виводі видно п'ять послідовних етапів, кожен із яких може завершитися окремо. Підставою стало порівняння виводів: в A.2 повністю зник блок TLS — отже, це окрема група; збій A.2 стався на рядку `Trying`, вже після успішного пошуку адреси — отже, з'єднання і пошук адреси теж різні групи. Змінити рішення змусило б, наприклад, якби перевірка сертифіката могла відбуватися без шифрування — тоді я розділила б групу 3 на дві.

**D.3. Питання, яке залишилося без відповіді**

> Чому TTL у двох запитах `dig` однаковий (3600), хоча між ними минуло 5 хвилин, а час відповіді зменшився з 265 мс до 28 мс? Швидша друга відповідь схожа на відповідь із кешу, але тоді TTL мав би зменшитися. Також не з'ясувала, чому з'єднання з neverssl.com на порт 80 не встановилося: проблема на боці сервера чи порт 80 блокується в моїй мережі.

---

## Використання штучного інтелекту

*Розділ обов'язковий. Заповнюється незалежно від того, чи використовувався ШІ. Детальні вимоги — у документі «Політика використання технологій штучного інтелекту».*

**Факт використання:** використано

**Установлений рівень для цієї роботи:** Р3 — ШІ як співвиконавець

**Фактичний рівень використання:** Р3

### Використані системи

| Система | Версія або модель | Період використання |
|---|---|---|
| Claude (Anthropic) | Claude Opus 5.5 | 19–20.11.2026 |

### Промпти

*Наводити дослівно, у тому вигляді, у якому запит було надано системі. Переказ не приймається.*

| № | Розділ роботи | Текст промпта |
|---|---|---|
| 1 | Підготовка середовища | це моя практична 1 чи я можу її виконати на мак |
| 2 | Частина A (оформлення виводів у звіті) | Я не розумію як переносити |
| 3 | Частина B, контрольні питання, висновки | дай мені всі видповіді на питання |

### Дії з отриманим результатом

| № промпта | Що перевірено | Що змінено | Що відхилено і чому |
|---|---|---|---|
| 1 | Що команди curl і dig працюють у терміналі macOS без встановлення | Команди виконано самостійно, виводи збережено у файли `raw/` | — |
| 2 | Що у звіт вставлено повні виводи без змін (звірено з файлами `raw/`) | Повторено запити `dig`, оскільки перший повторний запит було виконано без доменного імені | Ручне редагування виводів — виводи мають залишатися без змін |
| 3 | Відповіді звірено з власними виводами: кількість рядків A.1 перераховано, значення TTL, IP-адреси і тексти помилок перевірено за файлами | Формулювання відповідей і висновків переписано власними словами | Твердження, яких не видно у власних виводах |

### Підтвердження

Підтверджую, що всі наведені в цьому звіті виводи команд отримано мною особисто внаслідок фактичного виконання відповідних дій, а відомості цього розділу є повними та достовірними.

> Виводи `curl`, `dig` та інші артефакти не можуть бути згенеровані. Це стосується будь-якого рівня використання ШІ.

---

## Примітки виконавця

*(необов'язковий розділ: що не спрацювало, які команди довелося змінити, які виникли труднощі)*

> Запит A.2 до neverssl.com завершився тайм-аутом: з'єднання на порт 80 не встановилося. Перший повторний запит `dig` випадково виконано без доменного імені, тому обидва запити A.3 повторено о 12:08 та 12:13. Виводи вставлено у звіт без змін безпосередньо з файлів у папці `raw/`.