# Практична робота № 2

**Дисципліна:** Основи побудови інформаційних систем та мереж (ОК-13)

**Тема:** Структура повідомлень прикладного протоколу HTTP. Формування запиту вручну

| Поле | Значення |
|---|---|
| Студент | Кашутіна Юлія Петрівна |
| Група | ЗКН-21 |
| Номер варіанта | 14 |
| Індивідуальний домен | archlinux.org |
| «Чужий» домен для завдання A.3.1 (варіант ± 20) | suckless.org (варіант 29; згідно з методичкою ±15) |
| Середовище виконання | macOS |
| Дата виконання | 21.11.2026 |

---

## Частина A. Збір експериментальних даних

### Завдання A.1. Формування запиту вручну

**Команда:**

```
nc -c archlinux.org 80 | tee raw/a1-manual.txt
```

**Набраний запит:**

```
GET / HTTP/1.1
Host: archlinux.org
Connection: close

```

**Відповідь:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sat, 21 Nov 2026 15:28:23 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://archlinux.org/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

---

### Завдання A.2. Запит без поля `Host` у версії 1.1

**Команда:**

```
(printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n'; sleep 3) | nc archlinux.org 80
```

**Вивід:**

```
HTTP/1.1 400 Bad Request
Server: nginx
Date: Sat, 21 Nov 2026 15:31:10 GMT
Content-Type: text/html
Content-Length: 150
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

---

### Завдання A.3. Вплив поля `Host` на відповідь сервера

#### A.3.1. Чуже доменне ім'я в полі `Host`

**Команда:**

```
(printf 'GET / HTTP/1.1\r\nHost: suckless.org\r\nConnection: close\r\n\r\n'; sleep 3) | nc archlinux.org 80
```

**Вивід:**

```
HTTP/1.1 404 Not Found
Server: nginx
Date: Sat, 21 Nov 2026 15:31:13 GMT
Content-Type: text/html
Content-Length: 146
Connection: close

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

#### A.3.2. Неіснуюче ім'я в полі `Host`

**Команда:**

```
(printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n'; sleep 3) | nc archlinux.org 80
```

**Вивід:**

```
HTTP/1.1 404 Not Found
Server: nginx
Date: Sat, 21 Nov 2026 15:31:16 GMT
Content-Type: text/html
Content-Length: 146
Connection: close

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

#### A.3.3. Запит без поля `Host` у версії 1.0

**Команда:**

```
(printf 'GET / HTTP/1.0\r\n\r\n'; sleep 3) | nc archlinux.org 80
```

**Вивід:**

```
HTTP/1.1 404 Not Found
Server: nginx
Date: Sat, 21 Nov 2026 15:31:19 GMT
Content-Type: text/html
Content-Length: 146
Connection: close

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Зведення результатів наведено в **Додатку Д**.

---

### Завдання A.4. Два запити в одному з'єднанні

**Команда:**

```
(printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: archlinux.org\r\n\r\nGET / HTTP/1.1\r\nHost: archlinux.org\r\nConnection: close\r\n\r\n'; sleep 3) | nc archlinux.org 80
```

**Вивід:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sat, 21 Nov 2026 15:31:22 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: https://archlinux.org/opism-pr02-12345

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sat, 21 Nov 2026 15:31:22 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://archlinux.org/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

**Кількість отриманих відповідей:** 2

**Коди стану отриманих відповідей:** `301 Moved Permanently` (перша, з `Location: https://archlinux.org/opism-pr02-12345` і `Connection: keep-alive`) та `301 Moved Permanently` (друга, з `Location: https://archlinux.org/` і `Connection: close`)

---

### Завдання A.5. Запит за допомогою клієнтської програми

**Команда:**

```
curl -v --http1.1 http://archlinux.org/ -o /dev/null
```

**Вивід:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host archlinux.org:80 was resolved.
* IPv6: (none)
* IPv4: 209.126.35.79
*   Trying 209.126.35.79:80...
* Connected to archlinux.org (209.126.35.79) port 80
> GET / HTTP/1.1
> Host: archlinux.org
> User-Agent: curl/8.7.1
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Server: nginx
< Date: Sat, 21 Nov 2026 15:29:52 GMT
< Content-Type: text/html
< Content-Length: 162
< Connection: keep-alive
< Location: https://archlinux.org/
< 
{ [162 bytes data]

100   162  100   162    0     0    674      0 --:--:-- --:--:-- --:--:--   677
* Connection #0 to host archlinux.org left intact
```

---

### Завдання A.6. Запит через захищене з'єднання

**Ресурс, на якому виконано завдання:** `archlinux.org` (власний домен; у завданні A.1 отримано код 301)

**Підстава для використання резервного ресурсу (заповнюють за потреби):** не потрібна

**Команда:**

```
openssl s_client -connect archlinux.org:443 -servername archlinux.org -crlf -quiet 2>&1 | tee raw/a6-tls.txt
```

**Набраний запит:**

```
GET / HTTP/1.1
Host: archlinux.org
Connection: close

```

**Вивід:**

```
depth=4 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=3 C = US, O = Internet Security Research Group, CN = ISRG Root X2
verify return:1
depth=2 C = US, O = ISRG, CN = Root YE
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = YE1
verify return:1
depth=0 CN = archlinux.org
verify return:1
HTTP/1.1 200 OK
Server: nginx
Date: Sat, 21 Nov 2026 15:34:08 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 24764
Connection: close
Cache-Control: max-age=307
Content-Security-Policy: script-src 'self'; base-uri 'none'; default-src 'self'; form-action 'self'; frame-ancestors 'none'; img-src 'self' data:
ETag: "fdd358d181141fca9e6fe3eb83d9c69c"
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin
Cross-Origin-Opener-Policy: same-origin
X-Frame-Options: DENY
Vary: Cookie
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Cache-Status: HIT

<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="theme-color" content="#08C" />
    <title>Arch Linux</title>
    <link rel="stylesheet" type="text/css" href="/static/archlinux_common_style/navbar.css">
    <link rel="stylesheet" type="text/css" href="/static/archweb.css" media="screen" />
    <link rel="icon" type="image/png" href="/static/archlinux_common_style/favicon.png" />
    <link rel="shortcut icon" type="image/png" href="/static/archlinux_common_style/favicon.png" />
    <link rel="apple-touch-icon" href="/static/archlinux_common_style/apple-touch-icon-57x57.png" />
    <link rel="apple-touch-icon" sizes="72x72" href="/static/archlinux_common_style/apple-touch-icon-72x72.png" />
    <link rel="apple-touch-icon" sizes="114x114" href="/static/archlinux_common_style/apple-touch-icon-114x114.png" />
    <link rel="apple-touch-icon" sizes="144x144" href="/static/archlinux_common_style/apple-touch-icon-144x144.png" />
    <link rel="search" type="application/opensearchdescription+xml" href="/opensearch/packages/" title="Arch Linux Packages" />
    
<link rel="alternate" type="application/rss+xml" title="Arch Linux News Updates" href="/feeds/news/" />
<link rel="alternate" type="application/rss+xml" title="Arch Linux Package Updates" href="/feeds/packages/" />
<script type="text/javascript" src="/static/homepage.js" defer></script>
<link rel="me" href="https://fosstodon.org/@archlinux" title="Arch Linux Mastodon" />

</head>
<body class="">
    <header class="anb-home">
        <div id="archnavbar">
	<div id="logo"><a href="/" title="Return to the main page">Arch Linux</a></div>
	<div id="archnavbarmenu">
		<ul id="archnavbarlist">
			<li id="anb-home"><a href="/" title="Arch news, packages, projects and more">Home</a></li>
			<li id="anb-packages"><a href="/packages/" title="Arch Package Database">Packages</a></li>
			<li id="anb-forums"><a href="https://bbs.archlinux.org/" title="Community forums">Forums</a></li>
			<li id="anb-wiki"><a href="https://wiki.archlinux.org/" title="Community documentation">Wiki</a></li>
			<li id="anb-gitlab"><a href="https://gitlab.archlinux.org/archlinux" title="GitLab">GitLab</a></li>
			<li id="anb-security"><a href="https://security.archlinux.org/" title="Arch Linux Security Tracker">Security</a></li>
			<li id="anb-aur"><a href="https://aur.archlinux.org/" title="Arch Linux User Repository">AUR</a></li>
			<li id="anb-download"><a href="/download/" title="Get Arch Linux">Download</a></li>
		</ul>
	</div>
</div>

    </header>
    <div id="content">
        <div id="archdev-navbar">
            
        </div>
        
        
            <div id="content-left-wrapper">
                <div id="content-left">
                    

<div id="intro" class="box">
    <h2>A simple, lightweight distribution</h2>

    <p>You've reached the website for <strong>Arch Linux</strong>, a
    lightweight and flexible Linux® distribution that tries to Keep It
    Simple.</p>

    <p>Currently we have official packages optimized for the x86-64
    architecture. We complement our official package sets with a
    <a href="https://aur.archlinux.org/" title="Arch User Repository (AUR)">
        community-operated package repository</a> that grows in size and
    quality each and every day.</p>

    <p>Our strong community is diverse and helpful, and we pride ourselves
    on the range of skillsets and uses for Arch that stem from it. Please
    check out our <a href="https://bbs.archlinux.org/" title="Arch Forums">forums</a>
    and <a href="https://lists.archlinux.org/" title="Arch Mailing Lists">mailing lists</a>
    to get your feet wet. Also glance through our <a href="https://wiki.archlinux.org/"
        title="Arch Wiki">wiki</a>
    if you want to learn more about Arch.</p>

    <p class="readmore"><a href="/about/"
        title="Learn more about Arch Linux">Learn more...</a></p>
</div>

<div id="news">
    <h3>
        <a href="/news/" title="Browse the news archives">Latest News</a>
        <span class="arrow"></span>
    </h3>

    <a href="/feeds/news/" title="Arch News RSS Feed"
        class="rss-icon"><img src="/static/rss.svg" alt="RSS Feed" /></a>

    
    <h4>
        <a href="/news/update-to-php-8512-andor-php-legacy-8336-to-prevent-remote-code-execution/"
            title="View full article: Update to php ≥ 8.5.12 and/or php-legacy ≥ 8.3.36 to prevent remote code execution">Update to php ≥ 8.5.12 and/or php-legacy ≥ 8.3.36 to prevent remote code execution</a>
    </h4>
    <p class="timestamp">2026-10-21</p>
    <div class="article-content">
        <p>PHP versions prior to 8.5.12 and 8.3.36 are affected by <a href="https://www.cve.org/CVERecord?id=CVE-2026-109618">CVE-2026-109618</a> and <a href="https://www.cve.org/CVERecord?id=CVE-2026-109619">CVE-2026-109619</a>, which allow for remote code execution.</p>
<p>For any system publicly exposing a PHP application with the help of a prior version of PHP, we urge users to upgrade those systems to the latest available version of PHP (php ≥ 8.5.12 and php-legacy ≥ 8.3.36) immediately.</p>
<p>Edit: For further details and instructions, please refer to the Arch Linux Security Advisories <a href="https://security.archlinux.org/ASA-202610-1">ASA-202610-1</a> and <a href="https://security.archlinux.org/ASA-202610-2">ASA-202610-2</a>.</p>
        
    </div>
    
    <h4>
        <a href="/news/update-to-ca-certificates-mozilla-31311-1-to-prevent-man-in-the-middle-attacks/"
            title="View full article: Update to ca-certificates-mozilla ≥ 3.131.1-1 to prevent man-in-the-middle attacks">Update to ca-certificates-mozilla ≥ 3.131.1-1 to prevent man-in-the-middle attacks</a>
    </h4>
    <p class="timestamp">2026-10-21</p>
    <div class="article-content">
        <p>On 2026-10-20 it was revealed, that <a href="https://emudhra.we-have-a-problem.com/">several certificates of the eMudhra intermediate Certificate Authority have been compromised</a>.</p>
<p>The compromised certificates have been removed/invalidated with ca-certificates-mozilla ≥ 3.131.1-1.</p>
<p>Without an update to this central certificate bundle, systems are susceptible to man-in-the-middle attacks. As such, we urge everyone to upgrade their systems immediately.</p>
    </div>
    
    <h4>
        <a href="/news/mkinitcpio-42-requires-manual-intervention-for-tpm2-based-unlocking-of-luks-devices/"
            title="View full article: Mkinitcpio &gt;=42 requires manual intervention for TPM2-based unlocking of LUKS devices">Mkinitcpio &gt;=42 requires manual intervention for TPM2-based unlocking of LUKS devices</a>
    </h4>
    <p class="timestamp">2026-09-22</p>
    <div class="article-content">
        <p>Starting with package version <code>42-1</code>, the mkinitcpio systemd hook now includes <code>systemd-pcrosseparator.service</code> (as <a href="https://github.com/systemd/systemd/blob/1f9d56a7ab1d814044ac02939b42fd0efb99a8ab/NEWS#L69">intended by systemd v261</a>).</p>
<p>This affects the measurements of PCR values <code>0-7</code>, <code>9</code> and <code>12-14</code>. If you configured the systemd hook and unlock LUKS partitions, that depend on these values, you need to re-enroll the <a href="https://wiki.archlinux.org/title/Trusted_Platform_Module">TPM2</a> in use.</p>
<p>For guidance, please refer to <a href="https://man.archlinux.org/man/systemd-cryptenroll.1">systemd-cryptenroll(1)</a>, or the <a href="https://wiki.archlinux.org/title/Systemd-cryptenroll">ArchWiki article on systemd-cryptenroll</a>, when using pinned values. Refer to <a href="https://man.archlinux.org/man/core/systemd/systemd-pcrlock.8">systemd-pcrlock(8)</a>, when relying on custom policies and a disabled <code>systemd-pcrlock-make-policy.service</code>.</p>
    </div>
    
    <h4>
        <a href="/news/virtualbox-ext-vnc-7212-2-requires-manual-intervention/"
            title="View full article: virtualbox-ext-vnc &gt;= 7.2.12-2 requires manual intervention">virtualbox-ext-vnc &gt;= 7.2.12-2 requires manual intervention</a>
    </h4>
    <p class="timestamp">2026-07-21</p>
    <div class="article-content">
        <p>Previously, we installed its contents in a way that made <code>pacman</code> not aware of the files (using <code>VBoxManage extpack install ...</code> from an install script). To mitigate issues during upgrade, you can use one of the following methods:</p>
<ul>
<li>Uninstall <code>virtualbox-ext-vnc</code> before upgrading the system, then installing it again.</li>
<li>Run <code>VBoxManage extpack uninstall &#x27;VNC&#x27;</code> as root before upgrading the system.</li>
<li>Instruct <code>pacman</code> once to overwrite the existing files:</li>
</ul>
<p><code>pacman -Syu --overwrite &#x27;/usr/lib/virtualbox/ExtensionPacks/VNC/*&#x27;</code></p>
    </div>
    
    <h4>
        <a href="/news/active-aur-malicious-packages-incident/"
            title="View full article: Active AUR malicious packages incident">Active AUR malicious packages incident</a>
    </h4>
    <p class="timestamp">2026-06-12</p>
    <div class="article-content">
        <p>We are currently experiencing a high volume of malicious package adoptions and updates in the Arch User Repository.</p>
<p>We are actively working to track down existing malicious commits and attempting to prevent additional malicious commits from being pushed. While this is happening, and while we work to create a more permanent solution, users may see issues with the following:</p>
<ul>
<li>Creating new accounts on the AUR</li>
<li>Pushing package updates</li>
<li>Adopting or creating new packages</li>
</ul>
<p>We continue to encourage all users of AUR packages to review <em>all</em> PKGBUILD and install script changes when updating, especially …</p>
    </div>
    
    <h3>
        <a href="/news/"
            title="Browse the news archives">Older News</a>
        <span class="arrow"></span>
    </h3>
    <dl class="newslist">
    
        <dt>2026-06-04</dt>
        <dd>
            <a href="/news/arch-linux-2026-leader-election-results/"
                title="View full article: Arch Linux 2026 Leader Election Results">Arch Linux 2026 Leader Election Results</a>
        </dd>
    
    
        <dt>2026-05-25</dt>
        <dd>
            <a href="/news/breaking-changes-for-all-users-of-varnish-which-is-renamed-to-vinyl-cache/"
                title="View full article: Breaking changes for all users of `varnish`, which is renamed to `vinyl-cache`">Breaking changes for all users of `varnish`, which is renamed to `vinyl-cache`</a>
        </dd>
    
    
        <dt>2026-04-07</dt>
        <dd>
            <a href="/news/kea-1303-6-update-requires-manual-intervention/"
                title="View full article: kea &gt;= 1:3.0.3-6 update requires manual intervention">kea &gt;= 1:3.0.3-6 update requires manual intervention</a>
        </dd>
    
    
        <dt>2026-04-05</dt>
        <dd>
            <a href="/news/iptables-now-defaults-to-the-nft-backend/"
                title="View full article: iptables now defaults to the nft backend">iptables now defaults to the nft backend</a>
        </dd>
    
    
        <dt>2025-12-20</dt>
        <dd>
            <a href="/news/nvidia-590-driver-drops-pascal-support-main-packages-switch-to-open-kernel-modules/"
                title="View full article: NVIDIA 590 driver drops Pascal and lower support; main packages switch to Open Kernel Modules">NVIDIA 590 driver drops Pascal and lower support; main packages switch to Open Kernel Modules</a>
        </dd>
    
    
        <dt>2025-12-11</dt>
        <dd>
            <a href="/news/net-packages-may-require-manual-intervention/"
                title="View full article: .NET packages may require manual intervention">.NET packages may require manual intervention</a>
        </dd>
    
    
        <dt>2025-11-06</dt>
        <dd>
            <a href="/news/waydroid-154-3-update-may-require-manual-intervention/"
                title="View full article: waydroid &gt;= 1.5.4-3 update may require manual intervention">waydroid &gt;= 1.5.4-3 update may require manual intervention</a>
        </dd>
    
    
        <dt>2025-10-31</dt>
        <dd>
            <a href="/news/dovecot-24-requires-manual-intervention/"
                title="View full article: dovecot &gt;= 2.4 requires manual intervention">dovecot &gt;= 2.4 requires manual intervention</a>
        </dd>
    
    
        <dt>2025-08-21</dt>
        <dd>
            <a href="/news/recent-services-outages/"
                title="View full article: Recent service outages">Recent service outages</a>
        </dd>
    
    
        <dt>2025-08-04</dt>
        <dd>
            <a href="/news/zabbix-741-2-may-requires-manual-intervention/"
                title="View full article: zabbix &gt;= 7.4.1-2 may require manual intervention">zabbix &gt;= 7.4.1-2 may require manual intervention</a>
        </dd>
    </dl>
    
</div>


                </div>
            </div>
            <div id="content-right">
                

<div id="pkgsearch" class="widget">
    <form id="pkgsearch-form" method="get" action="/packages/">
        <fieldset>
            <label for="pkgsearch-field">Package Search:</label>
            <input id="pkgsearch-field" type="text" name="q" size="18" maxlength="200" autocomplete="off" />
        </fieldset>
    </form>
</div>

<div id="pkg-updates" class="widget box">
    <h3>Recent Updates <span class="more">(<a href="/packages/?sort=-last_update"
            title="Browse all of the latest packages">more</a>)</span></h3>

    <a href="/feeds/packages/" title="Arch Package Updates RSS Feed"
        class="rss-icon"><img src="/static/rss.svg" alt="RSS Feed" /></a>

    <table>
        
        <tr>
            <td class="pkg-name"><span class="extra">perl-data-dump 1.26-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/any/perl-data-dump/"
                    title="Details for perl-data-dump [extra]">any</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">python-pyreqwest 0.15.2-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/python-pyreqwest/"
                    title="Details for python-pyreqwest [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">par2cmdline 2.0.0-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/par2cmdline/"
                    title="Details for par2cmdline [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">python-cloudpickle 3.1.3-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/any/python-cloudpickle/"
                    title="Details for python-cloudpickle [extra]">any</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="core-testing testing">glibc 2.44+r83+g625054f30f0e-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/core-testing/x86_64/glibc/"
                    title="Details for glibc [core-testing]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">texstudio 4.9.10-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/texstudio/"
                    title="Details for texstudio [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">superfile 1.6.1-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/superfile/"
                    title="Details for superfile [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra-testing testing">cudnn-frontend 1.33.0-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra-testing/x86_64/cudnn-frontend/"
                    title="Details for cudnn-frontend [extra-testing]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra-testing testing">python-google-auth 2.63.0-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra-testing/any/python-google-auth/"
                    title="Details for python-google-auth [extra-testing]">any</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">python-yaml 6.0.4-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/python-yaml/"
                    title="Details for python-yaml [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra-testing testing">restic 0.20.0-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra-testing/x86_64/restic/"
                    title="Details for restic [extra-testing]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">graphviz 16.1.2-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/graphviz/"
                    title="Details for graphviz [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">python-fastapi 0.165.0-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/any/python-fastapi/"
                    title="Details for python-fastapi [extra]">any</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">nickle 2.113-1</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/nickle/"
                    title="Details for nickle [extra]">x86_64</a>
            </td>
        </tr>
        
        <tr>
            <td class="pkg-name"><span class="extra">hyprpaper 0.8.5-4</span></td>
            <td class="pkg-arch">
                <a href="/packages/extra/x86_64/hyprpaper/"
                    title="Details for hyprpaper [extra]">x86_64</a>
            </td>
        </tr>
        
    </table>
</div>



<div id="nav-sidebar" class="widget">
    <h4>Documentation</h4>
    <ul>
        <li><a href="https://wiki.archlinux.org/"
            title="Community documentation">Wiki</a></li>
        <li><a href="https://man.archlinux.org/"
            title="All manpages from Arch packages">Manual Pages</a></li>
        <li><a href="https://wiki.archlinux.org/title/Installation_guide"
            title="Installation guide">Installation Guide</a></li>
    </ul>

    <h4>Community</h4>
    <ul>
        <li><a href="https://lists.archlinux.org/"
            title="Community and developer mailing lists">Mailing Lists</a></li>
        <li><a href="https://wiki.archlinux.org/title/IRC_channels"
            title="Official and regional IRC communities">IRC Channels</a></li>
        <li><a href="https://planet.archlinux.org/"
            title="Arch in the blogosphere">Planet Arch</a></li>
        <li><a href="https://wiki.archlinux.org/title/International_communities"
            title="Arch communities in your native language">International Communities</a></li>
    </ul>

    <h4>Support</h4>
    <ul>
        <li><a href="/donate/" title="Help support Arch Linux">Donate</a></li>
        <li><a href="https://www.freewear.org/?page=list_items&amp;org=Archlinux"
            title="T-shirts">T-shirts via Freewear</a></li>
        <li><a href="https://www.hellotux.com/arch"
            title="T-shirts">T-shirts via HELLOTUX</a></li>
    </ul>

    <h4>Tools</h4>
    <ul>
        <li><a href="/mirrorlist/"
            title="Get a custom mirrorlist from our database">Mirrorlist Updater</a></li>
        <li><a href="/mirrors/"
            title="See a listing of all available mirrors">Mirror List</a></li>
        <li><a href="/mirrors/status/"
            title="Check the status of all known mirrors">Mirror Status</a></li>
    </ul>

    <h4>Development</h4>
    <ul>
        <li><a href="https://wiki.archlinux.org/title/Getting_involved"
            title="Getting involved">Getting involved</a></li>
        <li><a href="https://devblog.archlinux.page" title="Dev Blog">Dev Blog</a></li>
        <li><a href="https://gitlab.archlinux.org/archlinux/"
            title="Official Arch projects (git)">Projects in Git</a></li>
        <li><a href="https://rfc.archlinux.page"
            title="Arch Request for Comments">Request for Comments</a></li>
        <li><a href="/groups/"
            title="View the available package groups">Package Groups</a></li>
        <li><a href="/todo/"
            title="Developer Todo Lists">Todo Lists</a></li>
        <li><a href="/releng/releases/"
            title="Release Engineering ISO listing">ISO Release List</a></li>
        <li><a href="/visualize/"
            title="View visualizations">Visualizations</a></li>
        <li><a href="/packages/differences/"
            title="See differences in packages between available architectures">Differences Reports</a></li>
    </ul>

    <h4>People</h4>
    <ul>
        
        <li><a href="/people/developers/" title="More info about Developers">Developers</a></li>
        
        <li><a href="/people/package-maintainers/" title="More info about Package Maintainers">Package Maintainers</a></li>
        
        <li><a href="/people/support-staff/" title="More info about Support Staff">Support Staff</a></li>
        
        <li><a href="/people/developer-fellows/" title="More info about Developer Fellows">Developer Fellows</a></li>
        
        <li><a href="/people/package-maintainer-fellows/" title="More info about Package Maintainer Fellows">Package Maintainer Fellows</a></li>
        
        <li><a href="/people/support-staff-fellows/" title="More info about Support Staff Fellows">Support Staff Fellows</a></li>
        
        <li><a href="/master-keys/"
            title="Package/Database signing master keys">Signing Master Keys</a></li>
    </ul>

    <h4>More Resources</h4>
    <ul>
        <li><a href="https://wiki.archlinux.org/title/Arch_Linux_press_coverage"
            title="Arch Linux in the media">Press Coverage</a></li>
        <li><a href="/art/" title="Arch logos and other artwork for promotional use">Logos &amp; Artwork</a></li>
        <li><a href="/news/" title="News Archives">News Archives</a></li>
        <li><a href="/feeds/" title="Various RSS Feeds">RSS Feeds</a></li>
    </ul>
</div>

<div id="home-donate-button" class="widget">
    <a href="https://co.clickandpledge.com/Default.aspx?WID=47294">
        <img src="/static/click_and_pledge.png" alt="Donate via Click&amp;Pledge to Arch Linux" title="Donate via Click&amp;Pledge to Arch Linux"/>
    </a>
</div>

<div class="widget">
    <a href="https://www.hetzner.com" title="Dedicated Root Server, VPS &amp; Hosting - Hetzner Online GmbH">
        <img src="/static/hetzner_logo.png"
            title="" alt="Hetzner logo"/>
    </a>

    <a href="https://icons8.com/" title="Icons8">
        <img src="/static/icons8_logo.png"
            title="" alt="Icons8 logo"/>
    </a>
</div>


            </div>
        
        <div id="footer">
            <p>Copyright © 2002-2026 <a href="mailto:jvinet@zeroflux.org"
                title="Contact Judd Vinet">Judd Vinet</a>, <a href="mailto:aaron@archlinux.org"
                title="Contact Aaron Griffin">Aaron Griffin</a> and
                <a href="mailto:anthraxx@archlinux.org" title="Contact Levente Polyák">Levente Polyák</a>.</p>

            <p>The Arch Linux name and logo are recognized
            <a href="https://terms.archlinux.org/docs/trademark-policy/"
                title="Arch Linux Trademark Policy">trademarks</a>. Some rights reserved.</p>

            <p>The registered trademark Linux® is used pursuant to a sublicense from LMI,
            the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide basis.</p>
        </div>
    </div>
    <script type="application/ld+json">
    {
       "@context": "http://schema.org",
       "@type": "WebSite",
       "url": "https://archlinux.org/",
       "potentialAction": {
         "@type": "SearchAction",
         "target": "https://archlinux.org/packages/?q={search_term}",
         "query-input": "required name=search_term"
       }
    }
    </script>
    
</body>
</html>
```

---

## Частина B. Розбір полів заголовка

Розбирається відповідь, отримана в завданні A.1.

**Загальна кількість полів заголовка у відповіді:** 6

| № | Поле заголовка | Значення | Призначення (власне формулювання) | Походження: сервер / проміжний вузол / не визначено | Обґрунтування |
|---|---|---|---|---|---|
| 1 | `Server` | `nginx` | Назва програмного забезпечення вузла, який сформував відповідь | сервер (nginx на порту 80) | Тіло відповіді також містить підпис `<center>nginx</center>`, тобто сторінку з кодом 301 згенерував саме nginx |
| 2 | `Date` | `Sat, 21 Nov 2026 15:28:23 GMT` | Момент формування відповіді за часом сервера | сервер (nginx на порту 80) | Поле ставить той вузол, який створив відповідь; відповідь 301 створена nginx, а не взята з кешу |
| 3 | `Content-Type` | `text/html` | Тип даних у тілі відповіді — HTML-сторінка | сервер (nginx на порту 80) | Тіло — стандартна HTML-сторінка помилки nginx, тип описує саме її |
| 4 | `Content-Length` | `162` | Розмір тіла в байтах, щоб клієнт знав, де закінчується відповідь | сервер (nginx на порту 80) | Збігається з розміром стандартної сторінки nginx «301 Moved Permanently» (у A.4 і A.5 те саме значення 162) |
| 5 | `Connection` | `close` | Повідомляє, що після цієї відповіді з'єднання буде закрито | не визначено | Поле стосується лише поточного з'єднання між двома сусідніми вузлами; у відповідь на мій `Connection: close` його міг поставити будь-який вузол, що тримає TCP-з'єднання з клієнтом. За виводом не можна встановити, чи є перед nginx ще один вузол |
| 6 | `Location` | `https://archlinux.org/` | Нова адреса, за якою клієнт має повторити запит (перехід на HTTPS) | сервер (nginx на порту 80) | У A.4 значення `Location` повторює запитаний шлях (`/opism-pr02-12345`), тобто адресу формує правило переадресації nginx, а не застосунок сайту |

---

## Частина D. Висновки

**D.1.** Я не очікувала, що запит у версії 1.0 без поля `Host` (A.3.3) дасть `404 Not Found`, хоча в цій версії `Host` не обов'язковий. Також дивно, що на запит неіснуючої сторінки в A.4 сервер відповів не 404, а `301` і просто переадресував на HTTPS (`Location: https://archlinux.org/opism-pr02-12345`). А під час першої спроби A.1 сервер закрив з'єднання, бо я друкувала надто повільно.

**D.2.** Найважче було з полем `Connection`. Воно стосується лише з'єднання між двома сусідніми вузлами, тому незрозуміло, хто його поставив. До того ж у A.4 у першій відповіді воно `keep-alive`, а в другій `close`.

**D.3.** Чому без `Host` у версії 1.1 (A.2) сервер відповів `400`, а у версії 1.0 (A.3.3) — `404`? І чи є перед nginx ще якийсь вузол — у виводі A.1 цього не видно.

---

## Контрольні питання

**1.** Порожній рядок показує серверу, що заголовок запиту закінчився. До цього сервер чекає, бо не знає, чи будуть ще поля.

**2.** Сервер віддав правильну відповідь (`301` з `Location: https://archlinux.org/`) лише тоді, коли в `Host` було `archlinux.org` (A.1). Без `Host` у версії 1.1 — `400 Bad Request` (A.2). З чужим ім'ям (A.3.1), з неіснуючим ім'ям (A.3.2) і без `Host` у версії 1.0 (A.3.3) — однаковий `404 Not Found`. Поле `Host` потрібне, щоб сервер знав, який із сайтів на одній IP-адресі потрібен клієнту.

**3.** Прийшло дві відповіді, обидві `301`. Сервер на порту 80 не перевіряє, чи існує сторінка: навіть для неіснуючого шляху він просто переадресовує на HTTPS. Отже, його роль — перенаправляти на захищене з'єднання. Дві відповіді в одному з'єднанні означають, що браузер може завантажити багато файлів сторінки через одне з'єднання.

**4.** `curl` додав `User-Agent: curl/8.7.1` (яка програма робить запит) і `Accept: */*` (клієнт приймає будь-який тип даних).

**5.** У відповіді A.1 ознак проміжного вузла немає (немає `Via`, сторінка підписана `nginx`). Але це не означає, що його точно немає. У відповіді A.6 є `X-Cache-Status: HIT` — сторінку віддано з кешу, тобто перед сайтом працює кешувальний вузол.

**6.** Три рядки, про які не йшлося на лекції, наведено в **Додатку В**.

---

## Додаток В. Відповіді на питання 6

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `X-Cache-Status: HIT` | A.6 |
| 2 | `ETag: "fdd358d181141fca9e6fe3eb83d9c69c"` | A.6 |
| 3 | `depth=1 C = US, O = Let's Encrypt, CN = YE1` | A.6 |

---

## Додаток Д. Зведення результатів завдання A.3

**Вузол, з яким установлювалося з'єднання (у всіх пробах однаковий):** `archlinux.org`, порт 80

| Проба | Значення поля `Host` | Версія | Код стану | Обсяг тіла відповіді | Збігається з A.1 (так / ні) |
|---|---|---|---|---|---|
| A.1 (вихідна) | `archlinux.org` | 1.1 | 301 | 162 байти | — |
| A.2 | поле відсутнє | 1.1 | 400 | 150 байтів | ні |
| A.3.1 | `suckless.org` | 1.1 | 404 | 146 байтів | ні |
| A.3.2 | `opism-pr02.invalid` | 1.1 | 404 | 146 байтів | ні |
| A.3.3 | поле відсутнє | 1.0 | 404 | 146 байтів | ні |

**Висновок за таблицею:** Від проби до проби змінювалося лише значення поля `Host` (або його наявність) і версія протоколу, а з'єднання щоразу встановлювалося з тим самим вузлом. Свій сайт сервер розпізнав лише за `Host: archlinux.org`; для чужого, неіснуючого імені та для версії 1.0 без `Host` він повернув однакову сторінку 404, а запит версії 1.1 без `Host` відхилив як помилковий (400).

---

## Декларування використання технологій штучного інтелекту

Для цієї роботи встановлено **рівень Р3 — ШІ як співвиконавець**.

**Чи використовувалися технології ШІ під час виконання роботи:** так

| № | Інструмент (назва, версія) | Етап роботи | Дослівний текст запиту (промпту) | Як використано результат |
|---|---|---|---|---|
| 1 | Claude Opus 5.5 | Частина A | розпиши мені детальне виконання лаб 2 | Інструкції до виконання команд; команди виконано самостійно |
| 2 | Claude Opus 5.5 | Частина A | напеши детально я ще дуже плаваю у всьому цьому половини не розумію тому пиши покроково | Пояснення щодо особливостей `nc` на macOS |


**Підтвердження:** усі виводи команд, наведені в частині A, отримано внаслідок фактичного виконання команд на зазначеному індивідуальному домені.

---

## Примітки виконавця

На macOS утиліта `nc` завершувала роботу одразу після передавання запиту, не дочекавшись відповіді (файли виводу мали розмір 0), тому в неінтерактивних завданнях A.2–A.4 після `printf` додано `sleep 3`. Ключ CRLF у системній версії `nc` має вигляд `-c` (замість `-C` з методички). Перша спроба A.1 завершилася закриттям з'єднання сервером під час повільного ручного введення; запит повторено.

---

*ОПІСМ (ОК-13) · Практична робота № 2 · бланк звіту*
